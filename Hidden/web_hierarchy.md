# Odoo Module: web_hierarchy

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Web Hierarchy',
    'category': 'Hidden',
    'version': '1.0',
    'description':
        """
Odoo Web Hierarchy view
=======================

This module adds a new view called to be able to define a view to display
an organization such as an Organization Chart for employees for instance.
        """,
    'depends': ['web'],
    'assets': {
        'web.assets_backend': [
            'web_hierarchy/static/src/hierarchy.variables.scss',
            'web_hierarchy/static/src/**/*',
        ],
        'web.assets_web_dark': [
            ('before', 'web_hierarchy/static/src/hierarchy.variables.scss', 'web_hierarchy/static/src/**/*.variables.dark.scss'),
        ],
        'web.qunit_suite_tests': [
            'web_hierarchy/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\ir_actions.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ActWindowView(models.Model):
    _inherit = 'ir.actions.act_window.view'

    view_mode = fields.Selection(selection_add=[('hierarchy', 'Hierarchy')], ondelete={'hierarchy': 'cascade'})

```

## File: models\ir_ui_view.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from lxml import etree

from odoo import fields, models, _

HIERARCHY_VALID_ATTRIBUTES = {
    '__validate__',                     # ir.ui.view implementation detail
    'class',
    'js_class',
    'string',
    'create',
    'edit',
    'delete',
    'parent_field',
    'child_field',
    'icon',
    'draggable',
}

class View(models.Model):
    _inherit = 'ir.ui.view'

    type = fields.Selection(selection_add=[('hierarchy', "Hierarchy")])

    def _is_qweb_based_view(self, view_type):
        return super()._is_qweb_based_view(view_type) or view_type == "hierarchy"

    def _validate_tag_hierarchy(self, node, name_manager, node_info):
        if not node_info['validate']:
            return

        templates_count = 0
        for child in node.iterchildren(tag=etree.Element):
            if child.tag == 'templates':
                if not templates_count:
                    templates_count += 1
                else:
                    msg = _('Hierarchy view can contain only one templates tag')
                    self._raise_view_error(msg, child)
            elif child.tag != 'field':
                msg = _('Hierarchy child can only be field or template, got %s', child.tag)
                self._raise_view_error(msg, child)

        remaining = set(node.attrib) - HIERARCHY_VALID_ATTRIBUTES
        if remaining:
            msg = _(
                "Invalid attributes (%s) in hierarchy view. Attributes must be in (%s)",
                ','.join(remaining), ','.join(HIERARCHY_VALID_ATTRIBUTES),
            )
            self._raise_view_error(msg, node)

```

## File: models\models.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Base(models.AbstractModel):
    _inherit = 'base'

    @api.model
    def hierarchy_read(self, domain, fields, parent_field, child_field=None):
        if parent_field not in fields:
            fields.append(parent_field)
        records = self.search(domain)
        focus_record = self.env[self._name]
        fetch_child_ids_for_all_records = False
        if len(records) == 1:
            domain = [(parent_field, '=', records.id), ('id', '!=', records.id)]
            if records[parent_field]:
                focus_record = records
                records += focus_record[parent_field]
                domain = [('id', 'not in', records.ids), (parent_field, 'in', records.ids)]
            records += self.search(domain)
        elif not records:
            records = self.search([(parent_field, '=', False)])
        else:
            fetch_child_ids_for_all_records = True
        children_ids_per_record_id = {}
        if not child_field:
            children_ids_per_record_id = {
                record.id: child_ids
                for record, child_ids in self._read_group(
                    [(parent_field, 'in', records.ids if fetch_child_ids_for_all_records else (records - records[parent_field]).ids)],
                    (parent_field,),
                    ('id:array_agg',),
                )
            }
        result = records.read(fields)
        if children_ids_per_record_id:
            for record_data in result:
                if record_data['id'] in children_ids_per_record_id:
                    record_data['__child_ids__'] = children_ids_per_record_id[record_data['id']]
        return result

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_actions
from . import ir_ui_view
from . import models

```

## File: static\src\hierarchy_arch_parser.js

```javascript
/** @odoo-module */

import { visitXML } from "@web/core/utils/xml";
import { Field } from "@web/views/fields/field";
import { archParseBoolean, getActiveActions } from "@web/views/utils";

export class HierarchyArchParser {
    parse(xmlDoc, models, modelName) {
        const archInfo = {
            activeActions: getActiveActions(xmlDoc),
            draggable: false,
            icon: "fa-share-alt o_hierarchy_icon",
            parentFieldName: "parent_id",
            fieldNodes: {},
            templateDocs: {},
            xmlDoc,
        };
        const fieldNextIds = {};
        const fields = models[modelName];

        visitXML(xmlDoc, (node) => {
            if (node.hasAttribute("t-name")) {
                archInfo.templateDocs[node.getAttribute("t-name")] = node;
                return;
            }
            if (node.tagName === "hierarchy") {
                if (node.hasAttribute("parent_field")) {
                    const parentFieldName = node.getAttribute("parent_field");
                    if (!(parentFieldName in fields)) {
                        throw new Error(`The parent field set (${parentFieldName}) is not defined in the model (${modelName}).`);
                    } else if (fields[parentFieldName].type !== "many2one") {
                        throw new Error(`Invalid parent field, it should be a Many2One field.`);
                    } else if (fields[parentFieldName].relation !== modelName) {
                        throw new Error(`Invalid parent field, the co-model should be same model than the current one (expected: ${modelName}).`);
                    }
                    archInfo.parentFieldName = parentFieldName;
                }
                if (node.hasAttribute("child_field")) {
                    const childFieldName = node.getAttribute("child_field");
                    if (!(childFieldName in fields)) {
                        throw new Error(`The child field set (${childFieldName}) is not defined in the model (${modelName}).`);
                    } else if (fields[childFieldName].type !== "one2many") {
                        throw new Error(`Invalid child field, it should be a One2Many field.`);
                    } else if (fields[childFieldName].relation !== modelName) {
                        throw new Error(`Invalid child field, the co-model should be same model than the current one (expected: ${modelName}).`);
                    }
                    archInfo.childFieldName = childFieldName;
                }
                if (node.hasAttribute("draggable")) {
                    archInfo.draggable = archParseBoolean(node.getAttribute("draggable"));
                }
                if (node.hasAttribute("icon")) {
                    archInfo.icon = node.getAttribute("icon");
                }
            } else if (node.tagName === "field") {
                const fieldInfo = Field.parseFieldNode(node, models, modelName, "hierarchy");
                const name = fieldInfo.name;
                if (!(name in fieldNextIds)) {
                    fieldNextIds[name] = 0;
                }
                const fieldId = `${name}_${fieldNextIds[name]++}`;
                archInfo.fieldNodes[fieldId] = fieldInfo;
                node.setAttribute("field_id", fieldId);
            }
        });

        const cardDoc = archInfo.templateDocs["hierarchy-box"];
        if (!cardDoc) {
            throw new Error("Missing 'hierarchy-box' template.");
        }

        return archInfo;
    }
}

```

## File: static\src\hierarchy_card.js

```javascript
/** @odoo-module */

import { Component } from "@odoo/owl";

import { Field } from "@web/views/fields/field";
import { Record } from "@web/model/record";
import { ViewButton } from "@web/views/view_button/view_button";
import { useViewCompiler } from "@web/views/view_compiler";

import { HierarchyCompiler } from "./hierarchy_compiler";
import { getFormattedRecord } from "@web/views/kanban/kanban_record";

export class HierarchyCard extends Component {
    static components = {
        Record,
        Field,
        ViewButton,
    };
    static props = {
        node: Object,
        openRecord: Function,
        archInfo: Object,
        templates: Object,
        classNames: { type: String, optional: true },
    };
    static defaultProps = {
        classNames: "",
    };
    static template = "web_hierarchy.HierarchyCard";
    static Compiler = HierarchyCompiler;

    setup() {
        const { templates } = this.props;
        this.templates = useViewCompiler(this.constructor.Compiler, templates);
    }

    get classNames() {
        const classNames = [this.props.classNames];
        if (this.props.node.nodes.length) {
            classNames.push("o_hierarchy_node_unfolded");
        }
        return classNames.join(" ");
    }

    getRenderingContext(data) {
        const record = getFormattedRecord(data.record);
        return {
            context: this.props.node.context,
            JSON,
            luxon,
            record,
            __comp__: Object.assign(Object.create(this), { this: this }),
            __record__: data.record,
        };
    }

    onGlobalClick(ev) {
        if (ev.target.closest("button")) {
            return;
        }
        this.props.openRecord(this.props.node);
    }

    onClickArrowUp(ev) {
        this.props.node.fetchParentNode();
    }

    onClickArrowDown(ev) {
        if (this.props.node.nodes.length) {
            this.props.node.collapseChildNodes();
        } else {
            this.props.node.showChildNodes();
        }
    }
}

```

## File: static\src\hierarchy_card.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<templates>

    <t t-name="web_hierarchy.HierarchyCard">
        <div class="o_hierarchy_node_container d-flex flex-column" t-att-class="classNames" t-att-data-node-id="props.node.id">
            <div class="o_hierarchy_node_button_container w-100 d-flex justify-content-center">
                <button t-if="props.node.parentResId !== false and !props.node.parentNode"
                        name="hierarchy_search_parent_node"
                        class="btn p-0"
                        t-on-click.synthetic="onClickArrowUp"
                >
                    <i class="fa fa-chevron-up"/>
                </button>
            </div>
            <div class="o_hierarchy_node w-100 h-100 d-flex flex-column justify-content-between border mt-4"
                t-att-class="{
                    'border-bottom-0': props.node.nodes.length or props.node.canShowChildNodes,
                }"
                t-att-data-node-id="props.node.id"
                t-on-click.synthetic="onGlobalClick"
            >
                <div class="o_hierarchy_node_content">
                    <Record resModel="props.node.model.resModel"
                            resId="props.node.resId"
                            fields="props.node.model.fields"
                            activeFields="props.node.model.activeFields"
                            values="props.node.data"
                            t-slot-scope="data"
                    >
                        <t t-call="{{ templates['hierarchy-box'] }}" t-call-context="getRenderingContext(data)"/>
                    </Record>
                </div>
                <button t-if="props.node.nodes.length or props.node.canShowChildNodes"
                        name="hierarchy_search_subsidiaries"
                        class="o_hierarchy_node_button btn rounded-0 d-grid"
                        t-att-class="{
                            'btn-primary': !props.node.nodes.length,
                            'btn-secondary': props.node.nodes.length > 0,
                        }"
                        t-on-click.synthetic="onClickArrowDown"
                >
                    <t t-if="!props.node.nodes.length">
                        <span style="grid-column: 2;">
                            Unfold
                        </span>
                        <span class="text-end" style="grid-column: 3;">
                            <t t-out="props.node.childResIds.length"/>
                            <i class="fa ps-1" t-att-class="props.archInfo.icon"/>
                        </span>
                    </t>
                    <t t-else="">
                        <span style="grid-column: 2;">
                            Fold
                        </span>
                    </t>
                </button>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\hierarchy_compiler.js

```javascript
/** @odoo-module **/

import { KanbanCompiler } from "@web/views/kanban/kanban_compiler";

export class HierarchyCompiler extends KanbanCompiler {
    /**
     * @override
     * @param {Element} el
     * @param {Object} params
     * @returns {Element}
     */
    compileField(el, params) {
        const fieldName = el.getAttribute("name");
        return super.compileField(el, {
            ...(params || {}),
            recordExpr: "__record__",
            dataPointIdExpr: "__comp__.props.node.id",
            formattedValueExpr: `record['${fieldName}'].value`,
        });
    }

    compileButton(el, params) {
        return super.compileButton(el, {
            ...(params || {}),
            recordExpr: "__record__",
        });
    }
}

```

## File: static\src\hierarchy_controller.js

```javascript
/** @odoo-module */

import { Component, useRef } from "@odoo/owl";

import { useBus } from "@web/core/utils/hooks";
import { useModel } from "@web/model/model";
import { addFieldDependencies, extractFieldsFromArchInfo } from "@web/model/relational_model/utils";
import { CogMenu } from "@web/search/cog_menu/cog_menu";
import { Layout } from "@web/search/layout";
import { SearchBar } from "@web/search/search_bar/search_bar";
import { useSearchBarToggler } from "@web/search/search_bar/search_bar_toggler";
import { standardViewProps } from "@web/views/standard_view_props";
import { useViewButtons } from "@web/views/view_button/view_button_hook";

export class HierarchyController extends Component {
    static components = {
        Layout,
        CogMenu,
        SearchBar,
    };
    static props = {
        ...standardViewProps,
        Model: Function,
        Renderer: Function,
        buttonTemplate: String,
        archInfo: Object,
    };
    static template = "web_hierarchy.HierarchyView";

    setup() {
        this.rootRef = useRef("root");
        const { parentFieldName, childFieldName } = this.props.archInfo;
        const { activeFields, fields } = extractFieldsFromArchInfo(this.props.archInfo, this.props.fields);
        addFieldDependencies(activeFields, fields, [{ name: parentFieldName }]);
        this.model = useModel(this.props.Model, {
            resModel: this.props.resModel,
            activeFields,
            fields,
            parentFieldName,
            childFieldName,
        });
        useBus(
            this.model.bus,
            "update",
            () => {
                this.render(true);
            }
        );
        useViewButtons(this.model, this.rootRef, {
            beforeExecuteAction: this.beforeExecuteActionButton.bind(this),
            afterExecuteAction: this.afterExecuteActionButton.bind(this),
        });
        this.searchBarToggler = useSearchBarToggler();
    }

    async openRecord(node, mode) {
        const activeIds = this.model.root.resIds;
        this.props.selectRecord(node.resId, { activeIds, mode });
    }

    async beforeExecuteActionButton(clickParams) {}

    async afterExecuteActionButton(clickParams) {}
}

```

## File: static\src\hierarchy_controller.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<templates>
    <t t-name="web_hierarchy.HierarchyButtons">
        <div class="d-flex o_grid_buttons">
            <div class="me-2" t-if="props.archInfo.activeActions.create and (!props.archInfo.createInline or displayNoContent)">
                <button class="btn btn-primary o_hierarchy_button_add"
                        type="button"
                        t-on-click="props.createRecord">
                    New
                </button>
            </div>
        </div>
    </t>
    <t t-name="web_hierarchy.HierarchyView">
        <div t-attf-class="o_hierarchy_view {{ isMobile ? 'o_action_delegate_scroll' : '' }} {{ props.className }}" t-ref="root">
            <Layout className="(model.useSampleModel ? 'o_view_sample_data' : '') + ' d-flex'" display="props.display">
                <t t-set-slot="control-panel-additional-actions">
                    <CogMenu/>
                </t>
                <t t-set-slot="layout-buttons">
                    <t t-call="{{ props.buttonTemplate }}"/>
                </t>
                <t t-set-slot="layout-actions">
                    <SearchBar t-if="searchBarToggler.state.showSearchBar"/>
                </t>
                <t t-set-slot="control-panel-navigation-additional">
                    <t t-component="searchBarToggler.component" t-props="searchBarToggler.props"/>
                </t>
                <t t-set-slot="default" t-slot-scope="layout">
                    <t t-if="displayNoContent">
                        <t t-call="web.ActionHelper">
                            <t t-set="noContentHelp" t-value="props.info.noContentHelp"/>
                        </t>
                    </t>
                    <t t-component="props.Renderer"
                        model="model"
                        openRecord.bind="openRecord"
                        archInfo="props.archInfo"
                        templates="props.archInfo.templateDocs"
                    />
                </t>
            </Layout>
        </div>
    </t>
</templates>

```

## File: static\src\hierarchy_model.js

```javascript
/** @odoo-module */

import { Domain } from "@web/core/domain";
import { _t } from "@web/core/l10n/translation";
import { KeepLast, Mutex } from "@web/core/utils/concurrency";
import { pick } from "@web/core/utils/objects";
import { Model } from "@web/model/model";

let nodeId = 0;
let forestId = 0;
let treeId = 0;

/**
 * Get the id of the given many2one field value
 *
 * @param {false | [Number, string]} value many2one value
 * @returns {false | Number} id of the many2one
 */
function getIdOfMany2oneField(value) {
    return value && value[0];
}

export class HierarchyNode {
    /**
     * Constructor of hierarchy node stored in hierarchy tree
     *
     * @param {HierarchyModel} model
     * @param {Object} config
     * @param {Object} data
     * @param {HierarchyTree} tree
     * @param {HierarchyNode} parentNode
     * @param {Boolean} populateChildNodes
     */
    constructor(model, config, data, tree, parentNode = null, populateChildNodes = true) {
        this.id = nodeId++;
        this.data = data;
        this.parentNode = parentNode;
        this.tree = tree;
        this.model = model;
        this._config = config;
        tree.addNode(this);
        if (populateChildNodes) {
            this.populateChildNodes();
        }
    }

    /**
     * Get ancestor node
     *
     * @returns {HierarchyNode} ancestor node
     */
    get ancestorNode() {
        return this.parentNode ? this.ancestorNode : this;
    }

    /**
     * Is leaf?
     *
     * @returns {Boolean} False if the current node has node as child nodes, otherwise True.
     */
    get isLeaf() {
        return !this.nodes.length;
    }

    /**
     * Get forest of the current node
     *
     * @returns {HierarchyForest}
     */
    get forest() {
        return this.tree.forest;
    }

    /**
     * Get the resId of current node
     *
     * @returns {Number}
     */
    get resId() {
        return this.data.id;
    }

    /**
     * Get parent field name
     *
     * @returns {String}
     */
    get parentFieldName() {
        return this.model.parentFieldName;
    }

    /**
     * Get parent res id
     *
     * @returns {Number}
     */
    get parentResId() {
        return this.parentNode?.resId || getIdOfMany2oneField(this.data[this.parentFieldName]);
    }

    /**
     * Get child node res ids
     *
     * @returns {Number[]}
     */
    get childResIds() {
        return this.nodes.length ? this.nodes.map((node) => node.resId) : this.data[this.childFieldName]?.map((d) => typeof d === "number" ? d : d.id) || [];
    }

    /**
     * Get child field name
     *
     * @returns {String}
     */
    get childFieldName() {
        return this.model.childFieldName || this.model.defaultChildFieldName;
    }

    /**
     * Has child nodes?
     *
     * @returns {Boolean}
     */
    get hasChildren() {
        return this.nodes.length > 0 || this.data[this.childFieldName]?.length > 0;
    }

    /**
     * Can show parent node
     *
     * Knows if the parent node can be fetched and displayed inside the view
     *
     * @returns {Boolean} True if the current node has a parent node but it is not yet displayed and the data of the
     *                    current node is not already displayed in another node.
     */
    get canShowParentNode() {
        return Boolean(this.parentResId)
            && !this.parentNode
            && this.tree.forest.resIds.filter((resId) => resId === this.resId).length === 1;
    }


    /**
     * Can show child nodes
     *
     * Knows if the child nodes can be fetched and displayed inside the view
     *
     * @returns {Boolean} True if the current node has child nodes but they are not yet displayed and the data of the
     *                    current node is not already displayed in another node.
     */
    get canShowChildNodes() {
        return this.hasChildren
            && this.nodes.length === 0
            && this.tree.forest.resIds.filter((resId) => resId === this.resId).length === 1;
    }

    get descendantNodes() {
        const subNodes = [];
        if (!this.isLeaf) {
            subNodes.push(...this.nodes);
            for (const node of this.nodes) {
                if (node.descendantNodes.length) {
                    subNodes.push(...node.descendantNodes);
                }
            }
        }
        return subNodes;
    }

    /**
     * Get all descendants nodes parent resIds. If the current node has
     * descendants, the result also contains its resId.
     */
    get descendantsParentIds() {
        const descendantsParentIds = [];
        if (!this.isLeaf) {
            descendantsParentIds.push(this.resId);
            this.nodes.reduce((parentIds, node) => {
                if (!node.isLeaf) {
                    parentIds.push(...node.descendantsParentIds);
                }
                return parentIds;
            }, descendantsParentIds);
        }
        return descendantsParentIds;
    }

    /**
     * Get all descendants nodes resIds
     *
     * @returns {Number[]}
     */
    get allSubsidiaryResIds() {
        return this.descendantNodes.map((n) => n.resId);
    }

    /**
     * Populate child nodes
     *
     * Uses to create child nodes of the current one according to its data.
     */
    populateChildNodes() {
        this.nodes = [];
        const children = this.data[this.childFieldName] || [];
        if (
            children.length
            && children[0] instanceof Object
            && this.tree.forest.resIds.filter((resId) => resId === this.resId).length === 1
        ) {
            this.createChildNodes(children);
        }
    }

    /**
     * create child nodes
     *
     * @param {Object[]} childNodesData data of child nodes to generate
     */
    createChildNodes(childNodesData) {
        this.nodes = (childNodesData || this.data[this.childFieldName]).map(
            (childData) =>
                new HierarchyNode(
                    this.model,
                    this._config,
                    childData,
                    this.tree,
                    this
                )
        );
    }

    removeParentNode() {
        this.parentNode?.removeChildNode(this);
        this.parentNode = null;
        this.data[this.parentFieldName] = false;
    }

    /**
     * Fetch parent node
     */
    async fetchParentNode() {
        await this.model.fetchManager(this);
    }

    /**
     * Fetch child nodes
     */
    async showChildNodes() {
        await this.model.fetchSubordinates(this);
    }

    /**
     * Collapse child nodes
     *
     * Removes the descendant nodes of the current one and stores
     * the resIds of the child nodes in the data of the current one
     * to know it has child nodes to be able to show them again
     * when it is needed.
     */
    collapseChildNodes() {
        const childrenData = [];
        for (const childNode of this.nodes) {
            childNode.data[this.childFieldName] = childNode.childResIds;
            childrenData.push(childNode.data);
        }
        this.data[this.childFieldName] = childrenData;
        this.removeChildNodes();
        this.model.notify();
    }

    removeChildNode(node) {
        node.removeChildNodes();
        this.tree.removeNodes([node]);
        this.nodes = this.nodes.filter((n) => n.id !== node.id);
        this.data[this.childFieldName] = this.nodes.map((n) => n.data);
    }

    /**
     * Remove descendant nodes of the current one
     */
    removeChildNodes() {
        for (const childNode of this.nodes) {
            if (!childNode.isLeaf) {
                childNode.removeChildNodes();
            }
        }
        this.tree.removeNodes(this.nodes);
        this.nodes = [];
    }

    /**
     * Set parent node to the current node
     *
     * @param {HierarchyNode} node parent node to set
     */
    setParentNode(node) {
        this.parentNode = node;
        node.addChildNode(this);
        const tree = node.tree;
        if (tree.root === this) {
            tree.root = node;
        } else if (this.tree.root === this) {
            this.tree.removeRoot();
            this.setTree(node.tree);
        }
    }

    setTree(tree) {
        this.tree = tree;
        for (const childNode of this.nodes) {
            childNode.setTree(tree);
        }
    }

    /**
     * Adds child node to the current node
     *
     * @param {HierarchyNode} node child node to add
     */
    addChildNode(node) {
        this.nodes.push(node);
        this.data[this.childFieldName].push(node.data);
        this.tree.addNode(node);
    }
}

export class HierarchyTree {
    /**
     * Constructor
     *
     * @param {HierarchyModel} model
     * @param {Object} config config of the model
     * @param {Object} data root node data of the tree to create
     * @param {HierarchyForest} forest hierarchy forest containing the tree to create
     */
    constructor(model, config, data, forest) {
        this.id = treeId++;
        this.nodePerNodeId = {};
        this.forest = forest;
        if (data) {
            this.root = new HierarchyNode(model, config, data, this);
            this.forest.nodePerNodeId = {
                ...this.forest.nodePerNodeId,
                ...this.nodePerNodeId,
            };
        }
        this.model = model;
        this._config = config;
    }

    /**
     * Get node res ids inside the current tree
     *
     * @returns {Number}
     */
    get resIds() {
        return Object.values(this.nodePerNodeId).map((node) => node.resId);
    }

    /**
     * Add node inside the current tree
     *
     * @param {HierarchyNode} node node to add inside the current tree
     */
    addNode(node) {
        this.nodePerNodeId[node.id] = node;
        this.forest.addNode(node);
    }

    /**
     * Remove nodes inside the current tree
     *
     * @param {HierarchyNode} nodes nodes to remove
     */
    removeNodes(nodes) {
        const nodeIds = nodes.map((node) => node.id);
        this.nodePerNodeId = Object.fromEntries(
            Object.entries(this.nodePerNodeId)
                .filter(
                    ([nodeId,]) => !nodeIds.includes(Number(nodeId))
                )
            );
        this.forest.removeNodes(nodes);
    }

    removeRoot() {
        this.forest.removeTree(this);
    }
}

export class HierarchyForest {
    /**
     *
     * @param {HierarchyModel} model
     * @param {Object} config model config
     * @param {Object[]} data list of tree root nodes data
     */
    constructor(model, config, data) {
        this.id = forestId++;
        this.nodePerNodeId = {};
        this.trees = data.map((d) => new HierarchyTree(model, config, d, this));
        this.model = model;
        this._config = config;
    }

    /**
     * Get node res ids containing inside the current forest
     *
     * @returns {Number}
     */
    get resIds() {
        return Object.values(this.nodePerNodeId).map((node) => node.resId);
    }

    /**
     * Get root node of all trees inside the current forest
     *
     * @returns {HierarchyNode[]} root nodes
     */
    get rootNodes() {
        return this.trees.map((t) => t.root);
    }

    /**
     * Add a node inside the current forest
     *
     * @param {HierarchyNode} node node to add inside the current forest
     */
    addNode(node) {
        this.nodePerNodeId[node.id] = node;
    }

    /**
     * Removes nodes inside the current forest
     *
     * @param {HierarchyNode} nodes nodes to remove inside the current forest
     */
    removeNodes(nodes) {
        const nodeIds = nodes.map((node) => node.id);
        this.nodePerNodeId = Object.fromEntries(
            Object.entries(this.nodePerNodeId)
                .filter(
                    ([nodeId,]) => !nodeIds.includes(Number(nodeId))
                )
        );
    }

    addNewRootNode(node) {
        const tree = new HierarchyTree(this.model, this._config, null, this);
        tree.root = node;
        node.tree = tree;
        tree.addNode(node);
        for (const subNode of node.descendantNodes) {
            tree.addNode(subNode);
        }
        this.trees.push(tree);
    }

    removeTree(tree) {
        this.nodePerNodeId = Object.fromEntries(
            Object.entries(this.nodePerNodeId)
                .filter(
                    ([nodeId, ]) => !(nodeId in tree.nodePerNodeId)
                )
        );
        this.trees = this.trees.filter((t) => t.id !== tree.id);
    }
}

export class HierarchyModel extends Model {
    static services = ["notification"];

    setup(params, { notification }) {
        this.keepLast = new KeepLast();
        this.mutex = new Mutex();
        this.resModel = params.resModel;
        this.fields = params.fields;
        this.parentFieldName = params.parentFieldName;
        this.childFieldName = params.childFieldName;
        this.activeFields = params.activeFields;
        this.notification = notification;
        this.config = {
            domain: this.defaultDomain,
            isRoot: true,
        };
    }

    /**
     * Get parent field info
     *
     * @returns {Object} parent field info
     */
    get parentField() {
        return this.fields[this.parentFieldName];
    }

    /**
     * Get res ids of all nodes displayed in the view
     *
     * @returns {Number[]} resIds of all nodes displayed in the view
     */
    get resIds() {
        return this.root?.resIds || [];
    }

    /**
     * Get default child field name when no child field name is given to the view
     *
     * @returns {String} default child field name to use
     */
    get defaultChildFieldName() {
        return "__child_ids__";
    }

    /**
     * Get active fields name
     *
     * @returns {String[]} active fields name
     */
    get activeFieldNames() {
        return Object.keys(this.activeFields);
    }

    /**
     * Get default domain to use, when no domain is given in the config
     *
     * @returns {import("@web/src/core/domain").DomainListRepr} default domain
     */
    get defaultDomain() {
        return [[this.parentFieldName, '=', false]];
    }

    /**
     * Get fields to fetch
     * @returns {String[]} fields to fetch
     */
    get fieldsToFetch() {
        const fieldsToFetch = [
            ...this.activeFieldNames,
        ];
        if (this.childFieldName) {
            fieldsToFetch.push(this.childFieldName);
        }
        return fieldsToFetch;
    }

    get context() {
        return {
            bin_size: true,
            ...(this.config.context || {}),
        };
    }

    /**
     * Load the config and data for hierarchy view
     *
     * @param {Object} params params to use to load data of hierarchy view
     */
    async load(params = {}) {
        nodeId = forestId = treeId = 0;
        const config = this._getNextConfig(this.config, params);
        const data = await this.keepLast.add(this._loadData(config));
        this.root = this._createRoot(config, data);
        this.config = config;
        this.notify();
    }

    /**
     * @override
     * Each notify should specify a scroll target (default is to scroll to the
     * bottom).
     */
    notify(payload = { scrollTarget: "bottom" }) {
        super.notify();
        this.bus.trigger("hierarchyScrollTarget", payload);
    }

    /**
     * Fetch parent node of given node
     * @param {HierarchyNode} node node to fetch its parent node
     */
    async fetchManager(node) {
        if (this.root.trees.length > 1) { // reset the hierarchy
            const treeExpanded = this._findTreeExpanded();
            const resIdsToFetch = [node.parentResId, node.resId, ...node.allSubsidiaryResIds];
            if (treeExpanded && treeExpanded.root.id !== node.id && treeExpanded.root.parentResId === node.parentResId) {
                resIdsToFetch.push(...treeExpanded.root.allSubsidiaryResIds);
            }
            const config = {
                ...this.config,
                domain: ["|", [this.parentFieldName, "=", node.parentResId], ["id", "in", resIdsToFetch]],
            }
            const data = await this._loadData(config);
            this.root = this._createRoot(config, data);
            this.notify();
            return;
        }
        const managerData = await this.keepLast.add(this._fetchManager(node));
        if (managerData) {
            const parentNode = new HierarchyNode(this, this.config, managerData, node.tree, null, false);
            parentNode.createChildNodes();
            node.setParentNode(parentNode);
            this.notify();
        }
    }

    /**
     * Fetch child nodes of given node
     *
     * @param {HierarchyNode} node node to fetch its child nodes
     */
    async fetchSubordinates(node) {
        const childFieldName = this.childFieldName || this.defaultChildFieldName;
        const children = node.data[childFieldName];
        if (children.length) {
            const nodesToUpdate = [];
            if (!(children[0] instanceof Object)) {
                const allNodeResIds = this.root.resIds;
                const existingChildResIds = children.filter((childResId) => allNodeResIds.includes(childResId))
                if (existingChildResIds.length) { // special case with result found with the search view
                    for (const tree of this.root.trees) {
                        if (existingChildResIds.includes(tree.root.resId)) {
                            nodesToUpdate.push(tree.root);
                        }
                    }
                }
                const data = await this.keepLast.add(this._fetchSubordinates(node, existingChildResIds));
                if (data && data.length) {
                    node.data[childFieldName] = data;
                }
            }
            const nodeToCollapse = this._searchNodeToCollapse(node);
            if (nodeToCollapse && !nodesToUpdate.includes(nodeToCollapse)) {
                nodeToCollapse.collapseChildNodes();
            }
            node.populateChildNodes();
            for (const n of nodesToUpdate) {
                n.setParentNode(node);
            }
            this.notify();
        }
    }

    /**
     * Search node to collapse to be able to show the child nodes of node given in parameter
     *
     * @param {HierarchyNode} node node to show its child nodes.
     * @returns {HierarchyNode | null} node found to collapse
     */
    _searchNodeToCollapse(node) {
        const parentNode = node.parentNode;
        let nodeToCollapse = null;
        if (parentNode) {
            nodeToCollapse = parentNode.nodes.find((n) => n.nodes.length);
        } else {
            const treeExpanded = this._findTreeExpanded();
            if (treeExpanded) {
                nodeToCollapse = treeExpanded.root;
            }
        }
        return nodeToCollapse;
    }

    _findTreeExpanded() {
        return this.root.trees.find((t) => t.root.nodes.length);
    }

    /**
     * Get the next model config to use
     *
     * @param {Object} currentConfig current model config used
     * @param {Object} params new params
     * @returns {Object} new model config to use
     */
    _getNextConfig(currentConfig, params) {
        const config = Object.assign({}, currentConfig);
        config.context = "context" in params ? params.context : config.context;
        if ("domain" in params) {
            config.domain = params.domain;
            if (!params.domain.length) {
                if (config.context.hierarchy_res_id) {
                    config.domain = [["id", "=", config.context.hierarchy_res_id]];
                    delete config.context.hierarchy_res_id; // just needed for the first load.
                } else {
                    config.domain = this.defaultDomain;
                }
            }
        }
        return config;
    }

    /**
     * Load data for hierarchy view
     *
     * @param {Object} config model config
     * @returns {Object[]} main data for hierarchy view
     */
    async _loadData(config) {
        const result = await this.orm.call(
            this.resModel,
            "hierarchy_read",
            [config.domain, this.fieldsToFetch, this.parentFieldName, this.childFieldName],
            {
                context: this.context,
            },
        );
        const resultStringified = JSON.stringify(result);
        const recordsPerParentId = {};
        const recordPerId = {};
        for (const record of result) {
            recordPerId[record.id] = record;
            const parentId = getIdOfMany2oneField(record[this.parentFieldName]);
            if (!(parentId.toString() in recordsPerParentId)) {
                recordsPerParentId[parentId] = [];
            }
            recordsPerParentId[parentId].push(record);
        }
        const data = [];
        const recordIds = []; // to check if we have only one arborescence to display otherwise we display the result as the kanban view
        for (const [parentId, records] of Object.entries(recordsPerParentId)) {
            if (!parentId || !(parentId in recordPerId)) {
                data.push(...records);
            } else {
                const parentRecord = recordPerId[parentId];
                if (recordIds.includes(parentRecord.id)) {
                    return JSON.parse(resultStringified);
                }
                const ancestorId = getIdOfMany2oneField(parentRecord[this.parentFieldName]);
                if (ancestorId in recordsPerParentId) {
                    recordIds.push(...recordsPerParentId[ancestorId].map((r) => r.id));
                }
                parentRecord[this.childFieldName || this.defaultChildFieldName] = records;
            }
        }
        if (!data.length && result?.length) {
            data.push(recordPerId[Object.keys(recordsPerParentId)[0]]);
        }
        return data;
    }

    /**
     * Create forest
     *
     * @param {Object} config model config to use
     * @param {Object[]} data root data
     * @returns {HierarchyForest} forest hierarchy
     */
    _createRoot(config, data) {
        return new HierarchyForest(this, config, data);
    }

    /**
     * Fetch parent node and its children nodes data
     *
     * @param {HierarchyNode} node node to fetch its parent node
     * @returns {Object} the parent node data with children data inside childFieldName
     */
    async _fetchManager(node, exclude_node=true) {
        let domain = new Domain([
            "|",
                ["id", "=", node.parentResId],
                [this.parentFieldName, "=", node.parentResId],
        ]);
        if (exclude_node) {
            domain = Domain.and([
                domain,
                [["id", "!=", node.resId]],
            ])
        }
        const result = await this.orm.searchRead(
            this.resModel,
            domain.toList({}),
            this.fieldsToFetch,
            { context: this.context },
        );
        let managerData = {};
        const children = [];
        for (const data of result) {
            if (data.id === node.parentResId) {
                managerData = data;
            } else {
                children.push(data);
            }
        }
        if (!this.childFieldName) {
            if (children.length) {
                await this._fetchDescendants(children);
            }
        }
        managerData[this.childFieldName || this.defaultChildFieldName] = children;
        return managerData;
    }

    /**
     * Fetch children nodes data for a given node
     *
     * @param {HierarchyNode} node node to fetch its children nodes
     * @param {Array<number> | null} excludeResIds list of ids to exclude (because the nodes already exist)
     * @returns {Object[]} list of child node data
     */
    async _fetchSubordinates(node, excludeResIds = null) {
        let childrenResIds = node.data[this.childFieldName || this.defaultChildFieldName];
        if (excludeResIds) {
            childrenResIds = childrenResIds.filter((childResId) => !excludeResIds.includes(childResId));
        }
        const data = await this.orm.read(
            this.resModel,
            childrenResIds,
            this.fieldsToFetch,
            { context: this.context },
        )
        if (!this.childFieldName) {
            await this._fetchDescendants(data);
        }
        return data;
    }

    /**
     * fetch descendants nodes resIds to know if the child nodes have descendants
     *
     * @param {Object[]} childrenData child nodes data to fetch its descendants
     */
    async _fetchDescendants(childrenData) {
        const resIds = childrenData.map((d) => d.id);
        if (resIds.length) {
            const fetchChildren = await this.orm.readGroup(
                this.resModel,
                [[this.parentFieldName, "in", resIds]],
                ['id:array_agg'],
                [this.parentFieldName],
                { context: this.context },
            );
            const childIdsPerId = Object.fromEntries(
                fetchChildren.map((r) => [r[this.parentFieldName][0], r.id])
            );
            for (const d of childrenData) {
                if (d.id.toString() in childIdsPerId) {
                    d[this.defaultChildFieldName] = childIdsPerId[d.id.toString()];
                }
            }
        }
    }

    async updateParentNode(nodeId, { parentNodeId, parentResId }) {
        const node = this.root.nodePerNodeId[nodeId];
        const parentNode = parentNodeId ? this.root.nodePerNodeId[parentNodeId] : null;
        if (node) {
            const oldParentNode = node.parentNode;
            let fetchParentChildren = false;
            const descendantsParentIds = node.descendantsParentIds;
            let domain = new Domain([["id", "=", node.resId]]);
            if (oldParentNode) {
                domain = Domain.or([domain, [["id", "=", oldParentNode.resId]]]);
            }
            if (parentNode) {
                if (parentNode.resId === node.resId) {
                    this.notification.add(
                        _t("The parent record cannot be the record dragged."),
                        {
                            type: "danger",
                        }
                    );
                    return;
                } else if (node.allSubsidiaryResIds.includes(parentNode.resId)) {
                    this.notification.add(
                        _t("Cannot change the parent because it will cause a cyclic."),
                        {
                            type: "danger",
                        }
                    );
                    return;
                }
                domain = Domain.or([
                    domain,
                    [["id", "in", [parentNode.resId, node.resId]]],
                ]);
                if (parentNode.nodes.length === 0 && parentNode.childResIds.length > 0) {
                    fetchParentChildren = true;
                    domain = Domain.or([domain, [[this.parentFieldName, "=", parentNode.resId]]]);
                }
                if (node.id === node.tree.root.id) {
                    this.root.removeTree(node.tree);
                    node.tree = parentNode.tree;
                } else {
                    node.removeParentNode();
                }
            } else {
                node.removeParentNode();
                this.root.addNewRootNode(node);
            }
            this.notify({ scrollTarget: "none" });
            await this.mutex.exec(async () => {
                await this.orm.write(
                    this.resModel,
                    [node.resId],
                    { [this.parentFieldName]: parentResId || parentNode?.resId || false },
                    { context: this.context }
                );
            });
            if (descendantsParentIds.length) {
                domain = Domain.or([domain, [[this.parentFieldName, "in", descendantsParentIds]]]);
            }
            domain = domain.toList({});
            const descendants = {};
            if (domain.length) {
                const data = await this.orm.searchRead(
                    this.resModel,
                    domain,
                    this.fieldsToFetch,
                    { context: this.context },
                );
                const children = [];
                for (const d of data) {
                    const parentId = getIdOfMany2oneField(d[this.parentFieldName]);
                    if (d.id === node.resId) {
                        node.data = d;
                    } else if (d.id === oldParentNode?.resId) {
                        oldParentNode.data = d;
                    } else if (parentNode?.resId === d.id) {
                        const parentData = fetchParentChildren
                            ? {}
                            : pick(
                                  parentNode.data,
                                  this.childFieldName || this.defaultChildFieldName
                              );
                        parentNode.data = {
                            ...d,
                            ...parentData,
                        };
                    } else if (fetchParentChildren && parentId === parentNode.resId) {
                        children.push(d);
                    } else {
                        if (!(parentId in descendants)) {
                            descendants[parentId] = [];
                        }
                        descendants[parentId].push(d);
                    }
                }
                if (children.length) {
                    parentNode.data[this.childFieldName || this.defaultChildFieldName] = children;
                }
            }
            const treeExpanded = this._findTreeExpanded();
            if (parentNode) {
                if (treeExpanded && treeExpanded.id !== parentNode.tree.id) {
                    treeExpanded.root.collapseChildNodes();
                } else if (treeExpanded) {
                    const nodeToCollapse = this._searchNodeToCollapse(
                        parentNode.nodes.length ? parentNode.nodes[0] : parentNode
                    );
                    if (
                        nodeToCollapse &&
                        (descendants[node.resId]?.length || !parentNode.nodes.length)
                    ) {
                        nodeToCollapse.collapseChildNodes();
                    }
                }
                if (fetchParentChildren) {
                    parentNode.populateChildNodes();
                }
                node.setParentNode(parentNode);
            } else if (treeExpanded && descendants[node.resId]?.length) {
                treeExpanded.root.collapseChildNodes();
            }
            let descendantsParent = node;
            const handledParents = new Set([node.resId]);
            while (descendantsParent) {
                const parentId = descendantsParent.resId;
                const childNodesData = descendants[parentId];
                if (!childNodesData) {
                    break;
                }
                descendantsParent.data[this.childFieldName || this.defaultChildFieldName] =
                    childNodesData;
                descendantsParent.populateChildNodes();
                handledParents.add(descendantsParent.resId);
                descendantsParent = descendantsParent.nodes.find((n) => {
                    return !handledParents.has(n.resId) && n.resId in descendants;
                });
            }
            this.notify({ scrollTarget: nodeId });
        }
    }
}

```

## File: static\src\hierarchy_node_draggable.js

```javascript
/** @odoo-module */

import { onWillUnmount, reactive, useEffect, useExternalListener } from "@odoo/owl";
import { useThrottleForAnimation } from "@web/core/utils/timing";
import { pick } from "@web/core/utils/objects";
import { makeDraggableHook } from "@web/core/utils/draggable_hook_builder";

const hookParams = {
    name: "useHierarchyNodeDraggable",
    acceptedParams: {
        rows: [String],
    },
    defaultParams: {
        edgeScrolling: { speed: 20, threshold: 60 },
        rows: null,
    },
    onComputeParams({ ctx, params }) {
        // Row selector
        ctx.rowSelector = params.rows || null;
        if (ctx.rowSelector) {
            ctx.fullSelector = `${ctx.rowSelector} ${ctx.fullSelector}`;
        }
    },
    onDragStart(params) {
        const { ctx, addListener, callHandler } = params;

        const onElementPointerEnter = (ev) => {
            const element = ev.currentTarget;
            current.hierarchyElement = element;
            callHandler("onElementEnter", { element });
        };

        const onElementPointerLeave = (ev) => {
            const element = ev.currentTarget;
            current.hierarchyElement = null;
            callHandler("onElementLeave", { element });
        };

        const onRowPointerEnter = (ev) => {
            const row = ev.currentTarget;
            current.hierarchyRow = row;
            callHandler("onRowEnter", { row });
        };

        const onRowPointerLeave = (ev) => {
            const row = ev.currentTarget;
            current.hierarchyRow = null;
            callHandler("onRowLeave", { row });
        };

        const { ref, current, elementSelector, rowSelector } = ctx;

        for (const rowEl of ref.el.querySelectorAll(rowSelector)) {
            addListener(rowEl, "pointerenter", onRowPointerEnter);
            addListener(rowEl, "pointerleave", onRowPointerLeave);
        }

        for (const siblingEl of ref.el.querySelectorAll(elementSelector)) {
            if (siblingEl !== current.element) {
                addListener(siblingEl, "pointerenter", onElementPointerEnter);
                addListener(siblingEl, "pointerleave", onElementPointerLeave);
            }
        }

        return pick(current, "element", "row");
    },
    onDragEnd({ ctx }) {
        return pick(ctx.current, "element", "row", "hierarchyRow");
    },
    onDrop({ ctx }) {
        const { current } = ctx;
        const rowElement = current.hierarchyRow;
        const element = current.hierarchyElement;
        if ((rowElement && rowElement !== current.row) || element) {
            return {
                element: current.element,
                row: current.row,
                nextRow: rowElement && current.row !== rowElement ? rowElement : null,
                newParentNode: element,
            };
        }
    },
    onWillStartDrag({ ctx }) {
        const { current, rowSelector } = ctx;

        if (rowSelector) {
            current.row = current.element.closest(rowSelector);
        }

        return pick(current, "element", "row");
    },
};

export function useHierarchyNodeDraggable(params) {
    const setupHooks = {
        addListener: useExternalListener,
        setup: useEffect,
        teardown: onWillUnmount,
        throttle: useThrottleForAnimation,
        wrapState: reactive,
    }
    return makeDraggableHook({ ...hookParams, setupHooks })(params);
}

```

## File: static\src\hierarchy_renderer.js

```javascript
/** @odoo-module */

import { Component, useRef, onPatched } from "@odoo/owl";

import { _t } from "@web/core/l10n/translation";
import { useBus, useService } from "@web/core/utils/hooks";
import { scrollTo } from "@web/core/utils/scrolling";

import { HierarchyCard } from "./hierarchy_card";
import { useHierarchyNodeDraggable } from "./hierarchy_node_draggable";

export class HierarchyRenderer extends Component {
    static components = {
        HierarchyCard,
    };
    static props = {
        model: Object,
        openRecord: Function,
        archInfo: Object,
        templates: Object,
    };
    static template = "web_hierarchy.HierarchyRenderer";

    setup() {
        this.rendererRef = useRef("renderer");
        this.notification = useService("notification");
        if (this.canDragAndDropRecord) {
            useHierarchyNodeDraggable({
                ref: this.rendererRef,
                enable: this.draggable,
                elements: ".o_hierarchy_node_container",
                handle: ".o_hierarchy_node",
                rows: ".o_hierarchy_row",
                ignore: "button",
                onDragStart: ({ addClass, element }) => {
                    addClass(element, "o_hierarchy_dragged");
                    addClass(element.querySelector(".o_hierarchy_node"), "shadow");
                },
                onDragEnd: ({ removeClass, element, row, hierarchyRow }) => {
                    removeClass(element, "o_hierarchy_dragged");
                    if (row) {
                        removeClass(row, "o_hierarchy_hover");
                    }
                    if (hierarchyRow) {
                        removeClass(hierarchyRow, "o_hierarchy_hover");
                    }
                },
                onDrop: (params) => {
                    this.nodeDrop(params);
                },
                onElementEnter: ({ addClass, element }) => {
                    addClass(element, "o_hierarchy_hover");
                },
                onElementLeave: ({ removeClass, element }) => {
                    removeClass(element, "o_hierarchy_hover");
                },
                onRowEnter: ({ addClass, row }) => {
                    addClass(row, "o_hierarchy_hover");
                },
                onRowLeave: ({ removeClass, row }) => {
                    removeClass(row, "o_hierarchy_hover");
                },
            });
        }
        this.scrollTarget = "none";
        useBus(this.props.model.bus, "hierarchyScrollTarget", (ev) => {
            this.scrollTarget = ev.detail?.scrollTarget || "none";
        });
        onPatched(this.onPatched);
    }

    onPatched() {
        if (this.scrollTarget === "none") {
            return;
        }
        const row =
            this.scrollTarget === "bottom"
                ? this.rendererRef.el.querySelector(":scope .o_hierarchy_row:last-child")
                : this.rendererRef.el
                      .querySelector(
                          `:scope .o_hierarchy_node[data-node-id="${this.scrollTarget}"]`
                      )
                      ?.closest(".o_hierarchy_row");
        this.scrollTarget = "none";
        if (!row) {
            return;
        }
        scrollTo(row, { behavior: "smooth" });
    }

    get canDragAndDropRecord() {
        return this.draggable && !this.env.isSmall;
    }

    get draggable() {
        return this.props.archInfo.draggable;
    }

    get rows() {
        const rootNodes = this.props.model.root.rootNodes;
        const rows = [{ nodes: rootNodes }];
        const processNode = (node) => {
            if (!node.isLeaf) {
                rows.push({ parentNode: node, nodes: node.nodes});
                for (const subNode of node.nodes) {
                    processNode(subNode);
                }
            }
        };

        for (const node of this.props.model.root.rootNodes) {
            processNode(node);
        }

        return rows;
    }

    async nodeDrop({ element, row, nextRow, newParentNode }) {
        let parentNodeId, parentResId;
        if (newParentNode) {
            parentNodeId = newParentNode.dataset.nodeId;
        } else if (nextRow?.dataset.rowId !== row.dataset.rowId) {
            parentNodeId = nextRow.dataset.parentNodeId;
            if (!parentNodeId) {
                const nodes = this.rows[nextRow.dataset.rowId].nodes || [];
                if (nodes) {
                    parentNodeId = nodes[0].parentNode?.id;
                    if (!parentNodeId) {
                        parentResId = nodes[0].parentResId;
                        if (!nodes.every((node) => node.parentResId === parentResId)) {
                            this.notification.add(
                                _t("Impossible to update the parent node of the dragged node because no parent has been found."),
                                {
                                    type: "danger",
                                }
                            );
                            return;
                        }
                    }
                }
            }
        }
        await this.props.model.updateParentNode(element.dataset.nodeId, { parentResId, parentNodeId });
    }
}

```

## File: static\src\hierarchy_renderer.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<templates>

    <t t-name="web_hierarchy.HierarchyRenderer">
        <div class="o_hierarchy_renderer w-100 d-flex justify-content-center" t-ref="renderer">
            <div class="o_hierarchy_container d-flex flex-column px-3 pt-4 mw-xl-75">
                <t t-foreach="rows" t-as="row" t-key="row_index">
                    <t t-set="previousRow" t-value="!row_first ? rows[row_index - 1] : null"/>
                    <t t-if="!row_first">
                        <div t-if="row.parentNode and previousRow.nodes.length > 1"
                            class="o_hierarchy_parent_node_container d-flex justify-content-center"
                        >
                            <span t-esc="row.parentNode.data.name" />
                        </div>
                        <div class="o_hierarchy_separator d-flex">
                            <div class="o_hierarchy_line_part o_hierarchy_line_left"></div>
                            <div class="o_hierarchy_line_part o_hierarchy_line_right"></div>
                        </div>
                    </t>
                    <div class="o_hierarchy_row row justify-content-center flex-wrap row-cols-2 row-cols-lg-5 g-2 g-lg-3"
                        t-att-class="{ 'pb-4': row_last }" t-att-data-parent-node-id="row.parentNode?.id"
                        t-att-data-row-id="row_index"
                    >
                        <t t-foreach="row.nodes" t-as="node" t-key="node.id">
                            <HierarchyCard
                                node="node"
                                openRecord="props.openRecord"
                                archInfo="props.archInfo"
                                templates="props.templates"
                            />
                        </t>
                    </div>
                </t>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\hierarchy_view.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { HierarchyArchParser } from "./hierarchy_arch_parser";
import { HierarchyController } from "./hierarchy_controller";
import { HierarchyModel } from "./hierarchy_model";
import { HierarchyRenderer } from "./hierarchy_renderer";

export const hierarchyView = {
    type: "hierarchy",
    display_name: _t("Hierarchy"),
    icon: "fa fa-share-alt o_hierarchy_icon",
    isMobileFriendly: false,
    multiRecord: true,
    ArchParser: HierarchyArchParser,
    Controller: HierarchyController,
    Model: HierarchyModel,
    Renderer: HierarchyRenderer,
    buttonTemplate: "web_hierarchy.HierarchyButtons",
    searchMenuTypes: [],

    props: (genericProps, view) => {
        const { ArchParser, Model, Renderer, buttonTemplate: viewButtonTemplate } = view;
        const { arch, relatedModels, resModel, buttonTemplate } = genericProps;
        return {
            ...genericProps,
            archInfo: new ArchParser().parse(arch, relatedModels, resModel),
            buttonTemplate: buttonTemplate || viewButtonTemplate,
            Model,
            Renderer,
        };
    }
}

registry.category("views").add("hierarchy", hierarchyView);

```

