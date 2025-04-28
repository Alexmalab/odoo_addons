# Odoo Module: l10n_my_edi

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Malaysia - E-invoicing',
    'countries': ['my'],
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'icon': '/account/static/description/l10n.png',
    "summary": "E-invoicing using MyInvois",
    'description': """
    This modules allows the user to send their invoices to the MyInvois system.
    """,
    # The export does not depend on the pint format, but we need to reuse the fields defined there.
    'depends': ['l10n_my', 'l10n_my_ubl_pint', 'account_edi_proxy_client'],
    'data': [
        'data/ir_cron.xml',
        'data/l10n_my_edi.industry_classification.csv',
        'data/my_ubl_templates.xml',
        'security/ir.model.access.csv',
        'views/account_move_view.xml',
        'views/account_tax_view.xml',
        'views/l10n_my_edi_industrial_classification_views.xml',
        'views/product_template_view.xml',
        'views/res_company_view.xml',
        'views/res_config_settings_view.xml',
        'views/res_partner_view.xml',
        'wizard/l10n_my_edi_status_update_wizard.xml',
    ],
    'installable': True,
    'license': 'LGPL-3'
}

```

## File: data\ir_cron.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>
        <record id="ir_cron_myinvois_sync" model="ir.cron">
            <field name="name">MyInvois: Synchronization</field>
            <field name="model_id" ref="account.model_account_move"/>
            <field name="state">code</field>
            <field name="code">model._cron_l10n_my_edi_synchronize_myinvois()</field>
            <field name="interval_number">1</field>
            <field name="interval_type">hours</field>
            <field name="nextcall" eval="(DateTime.now() + timedelta(hours=1))"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_my_edi.industry_classification.csv

```csv
"id","code","name"
"class_00000","00000","NOT APPLICABLE"
"class_01111","01111","Growing of maize"
"class_01112","01112","Growing of leguminous crops"
"class_01113","01113","Growing of oil seeds"
"class_01119","01119","Growing of other cereals n.e.c."
"class_01120","01120","Growing of paddy"
"class_01131","01131","Growing of leafy or stem vegetables"
"class_01132","01132","Growing of fruits bearing vegetables"
"class_01133","01133","Growing of melons"
"class_01134","01134","Growing of mushrooms and truffles"
"class_01135","01135","Growing of vegetables seeds, except beet seeds"
"class_01136","01136","Growing of other vegetables"
"class_01137","01137","Growing of sugar beet"
"class_01138","01138","Growing of roots, tubers, bulb or tuberous vegetables"
"class_01140","01140","Growing of sugar cane"
"class_01150","01150","Growing of tobacco"
"class_01160","01160","Growing of fibre crops"
"class_01191","01191","Growing of flowers"
"class_01192","01192","Growing of flower seeds"
"class_01193","01193","Growing of sago (rumbia)"
"class_01199","01199","Growing of other non-perennial crops n.e.c."
"class_01210","01210","Growing of grapes"
"class_01221","01221","Growing of banana"
"class_01222","01222","Growing of mango"
"class_01223","01223","Growing of durian"
"class_01224","01224","Growing of rambutan"
"class_01225","01225","Growing of star fruit"
"class_01226","01226","Growing of papaya"
"class_01227","01227","Growing of pineapple"
"class_01228","01228","Growing of pitaya (dragon fruit)"
"class_01229","01229","Growing of other tropical and subtropical fruits n.e.c."
"class_01231","01231","Growing of pomelo"
"class_01232","01232","Growing of lemon and limes"
"class_01233","01233","Growing of tangerines and mandarin"
"class_01239","01239","Growing of other citrus fruits n.e.c."
"class_01241","01241","Growing of guava"
"class_01249","01249","Growing of other pome fruits and stones fruits n.e.c."
"class_01251","01251","Growing of berries"
"class_01252","01252","Growing of fruit seeds"
"class_01253","01253","Growing of edible nuts"
"class_01259","01259","Growing of other tree and bush fruits"
"class_01261","01261","Growing of oil palm (estate)"
"class_01262","01262","Growing of oil palm (smallholdings)"
"class_01263","01263","Growing of coconut (estate and smallholdings)"
"class_01269","01269","Growing of other oleaginous fruits n.e.c."
"class_01271","01271","Growing of coffee"
"class_01272","01272","Growing of tea"
"class_01273","01273","Growing of cocoa"
"class_01279","01279","Growing of other beverage crops n.e.c."
"class_01281","01281","Growing of pepper (piper nigrum)"
"class_01282","01282","Growing of chilies and pepper (capsicum spp.)"
"class_01283","01283","Growing of nutmeg"
"class_01284","01284","Growing of ginger"
"class_01285","01285","Growing of plants used primarily in perfumery, in pharmacy or for insecticidal, fungicidal or similar purposes"
"class_01289","01289","Growing of other spices and aromatic crops n.e.c."
"class_01291","01291","Growing  of rubber trees (estate)"
"class_01292","01292","Growing of rubber trees (smallholdings)"
"class_01293","01293","Growing of trees for extraction of sap"
"class_01294","01294","Growing of nipa palm"
"class_01295","01295","Growing of areca"
"class_01296","01296","Growing of roselle"
"class_01299","01299","Growing of other perennial crops n.e.c."
"class_01301","01301","Growing of plants for planting"
"class_01302","01302","Growing of plants for ornamental purposes"
"class_01303","01303","Growing of live plants for bulbs, tubers and roots; cuttings and slips; mushroom spawn"
"class_01304","01304","Operation of tree nurseries"
"class_01411","01411","Raising, breeding and production of cattle or buffaloes"
"class_01412","01412","Production of raw milk from cows or buffaloes"
"class_01413","01413","Production of bovine semen"
"class_01420","01420","Raising and breeding of horses, asses, mules or hinnes"
"class_01430","01430","Raising and breeding of camels (dromedary) and camelids"
"class_01441","01441","Raising, breeding and production of sheep and goats"
"class_01442","01442","Production of raw sheep or goat’s milk"
"class_01443","01443","Production of raw wool"
"class_01450","01450","Raising, breeding and production of swine/pigs"
"class_01461","01461","Raising, breeding and production of chicken, broiler"
"class_01462","01462","Raising, breeding and production of ducks"
"class_01463","01463","Raising, breeding and production of geese"
"class_01464","01464","Raising, breeding and production of quails"
"class_01465","01465","Raising and breeding of other poultry n.e.c."
"class_01466","01466","Production of chicken eggs"
"class_01467","01467","Production of duck eggs"
"class_01468","01468","Production of other poultry eggs n.e.c."
"class_01469","01469","Operation of poultry hatcheries"
"class_01491","01491","Raising, breeding and production of semi-domesticated"
"class_01492","01492","Production of fur skins, reptile or bird’s skin from ranching operation"
"class_01493","01493","Operation of worm farms, land mollusc farms, snail farms"
"class_01494","01494","Raising of silk worms and production of silk worm cocoons"
"class_01495","01495","Bee keeping and production of honey and beeswax"
"class_01496","01496","Raising and breeding of pet animals"
"class_01497","01497","Raising and breeding of swiflet"
"class_01499","01499","Raising of diverse/other animals n.e.c."
"class_01500","01500","Mixed Farming"
"class_01610","01610","Agricultural activities for crops production on a fee or contract basis"
"class_01620","01620","Agricultural activities for animal production on a fee or contract basis"
"class_01631","01631","Preparation of crops for primary markets"
"class_01632","01632","Preparation of tobacco leaves"
"class_01633","01633","Preparation of cocoa beans"
"class_01634","01634","Sun-drying of fruits and vegetables"
"class_01640","01640","Seed processing for propagation"
"class_01701","01701","Hunting and trapping on a commercial basis"
"class_01702","01702","Taking of animals (dead or alive)"
"class_02101","02101","Planting, replanting, transplanting, thinning and conserving of forests and timber tracts"
"class_02102","02102","Growing of coppice, pulpwood and fire wood"
"class_02103","02103","Operation of forest tree nurseries"
"class_02104","02104","Collection and raising of wildings (peat swamp forest tree species)"
"class_02105","02105","Forest plantation"
"class_02201","02201","Production of round wood for forest-based manufacturing industries"
"class_02202","02202","Production of round wood used in an unprocessed form"
"class_02203","02203","Production of charcoal in the forest (using traditional methods)"
"class_02204","02204","Rubber wood logging"
"class_02301","02301","Collection of rattan, bamboo"
"class_02302","02302","Bird’s nest collection"
"class_02303","02303","Wild sago palm collection"
"class_02309","02309","Gathering of non-wood forest products n.e.c."
"class_02401","02401","Carrying out part of the forestry and forest plantation operation on a fee or contract basis for forestry service activities"
"class_02402","02402","Carrying out part of the forestry operation on a fee or contract basis for logging service activities"
"class_03111","03111","Fishing on a commercial basis in ocean and coastal waters"
"class_03112","03112","Collection of marine crustaceans and molluscs"
"class_03113","03113","Taking of aquatic animals: sea squirts, tunicates, sea urchins"
"class_03114","03114","Activities of vessels engaged both in fishing and in processing and preserving of fish"
"class_03115","03115","Gathering of other marine organisms and materials (natural pearls, sponges, coral and algae)"
"class_03119","03119","Marine fishing n.e.c."
"class_03121","03121","Fishing on a commercial basis in inland waters"
"class_03122","03122","Taking of freshwater crustaceans and molluscs"
"class_03123","03123","Taking of freshwater aquatic animals"
"class_03124","03124","Gathering of freshwater flora and fauna"
"class_03129","03129","Freshwater fishing n.e.c."
"class_03211","03211","Fish farming in sea water"
"class_03212","03212","Production of bivalve spat (oyster, mussel), lobster lings, shrimp post-larvae, fish fry and fingerlings"
"class_03213","03213","Growing of laver and other edible seaweeds"
"class_03214","03214","Culture of crustaceans, bivalves, other molluscs and other aquatic animals in sea water"
"class_03215","03215","Aquaculture activities in brackish water"
"class_03216","03216","Aquaculture activities in salt water filled tanks or reservoirs"
"class_03217","03217","Operation of  hatcheries (marine)"
"class_03218","03218","Operation of marine worm farms for fish feed"
"class_03219","03219","Marine aquaculture n.e.c."
"class_03221","03221","Fish farming in freshwater"
"class_03222","03222","Shrimp farming in freshwater"
"class_03223","03223","Culture of freshwater crustaceans, bivalves, other molluscs and other aquatic animals"
"class_03224","03224","Operation of hatcheries (freshwater)"
"class_03225","03225","Farming of frogs"
"class_03229","03229","Freshwater aquaculture n.e.c."
"class_05100","05100","Mining of hard coal"
"class_05200","05200","Mining of lignite (brown coal)"
"class_06101","06101","Extraction of crude petroleum oils"
"class_06102","06102","Extraction of bituminous or oil shale and tar sand"
"class_06103","06103","Production of crude petroleum from bituminous shale and sand"
"class_06104","06104","Processes to obtain crude oils"
"class_06201","06201","Production of crude gaseous hydrocarbon (natural gas)"
"class_06202","06202","Extraction of condensates"
"class_06203","06203","Draining and separation of liquid hydrocarbon fractions"
"class_06204","06204","Gas desulphurization"
"class_06205","06205","Mining of hydrocarbon liquids, obtain through liquefaction or pyrolysis"
"class_07101","07101","Mining of ores valued chiefly for iron content"
"class_07102","07102","Beneficiation and agglomeration of iron ores"
"class_07210","07210","Mining of uranium and thorium ores"
"class_07291","07291","Mining of tin ores"
"class_07292","07292","Mining of copper"
"class_07293","07293","Mining of bauxite (aluminium)"
"class_07294","07294","Mining of ilmenite"
"class_07295","07295","Mining of gold"
"class_07296","07296","Mining of silver"
"class_07297","07297","Mining of platinum"
"class_07298","07298","Amang retreatment"
"class_07299","07299","Mining of other non-ferrous metal ores n.e.c."
"class_08101","08101","Quarrying, rough trimming and sawing of monumental and building stone such as marble, granite (dimension stone), sandstone"
"class_08102","08102","Quarrying, crushing and breaking of limestone"
"class_08103","08103","Mining of gypsum and anhydrite"
"class_08104","08104","Mining of chalk and uncalcined dolomite"
"class_08105","08105","Extraction and dredging of industrial sand, sand for construction and gravel"
"class_08106","08106","Breaking and crushing of stone and gravel"
"class_08107","08107","Quarrying of sand"
"class_08108","08108","Mining of clays, refractory clays and kaolin"
"class_08109","08109","Quarrying, crushing and breaking of granite"
"class_08911","08911","Mining of natural phosphates"
"class_08912","08912","Mining of natural potassium salts"
"class_08913","08913","Mining of native sulphur"
"class_08914","08914","Extraction and preparation of pyrites and pyrrhotite, except roasting"
"class_08915","08915","Mining of natural barium sulphate and carbonate (barytes and witherite)"
"class_08916","08916","Mining of natural borates, natural magnesium sulphates (kieserite)"
"class_08917","08917","Mining of earth colours, fluorspar and other minerals valued chiefly as a source of chemicals"
"class_08918","08918","Guano mining"
"class_08921","08921","Peat digging"
"class_08922","08922","Peat agglomeration"
"class_08923","08923","Preparation of peat to improve quality or facilitate transport or storage"
"class_08931","08931","Extraction of salt from underground"
"class_08932","08932","Salt production by evaporation of sea water or other saline waters"
"class_08933","08933","Crushing, purification and refining of salt by the producer"
"class_08991","08991","Mining and quarrying of abrasive materials"
"class_08992","08992","Mining and quarrying of asbestos"
"class_08993","08993","Mining and quarrying of siliceous fossil meals"
"class_08994","08994","Mining and quarrying of natural graphite"
"class_08995","08995","Mining and quarrying of steatite (talc)"
"class_08996","08996","Mining and quarrying of gemstones"
"class_08999","08999","Other mining and quarrying n.e.c."
"class_09101","09101","Oil and gas extraction service activities provided on a fee or contract basis"
"class_09102","09102","Oil and gas field fire fighting services"
"class_09900","09900","Support activities for other mining and quarrying"
"class_10101","10101","Processing and preserving of meat and production of meat products"
"class_10102","10102","Processing and preserving of poultry and poultry products"
"class_10103","10103","Production of hides and skins originating from slaughterhouses"
"class_10104","10104","Operation of slaughterhouses engaged in killing, houses dressing or packing meat"
"class_10109","10109","Processing and preserving of meat n.e.c."
"class_10201","10201","Canning of fish, crustaceans and mollusks"
"class_10202","10202","Processing, curing and preserving of fish, crustacean and molluscs"
"class_10203","10203","Production of fish meals for human consumption or animal feed"
"class_10204","10204","Production of keropok including keropok lekor"
"class_10205","10205","Processing of seaweed"
"class_10301","10301","Manufacture of fruits and vegetable food products"
"class_10302","10302","Manufacture of fruit and vegetable juices"
"class_10303","10303","Pineapple canning"
"class_10304","10304","Manufacture of jams, marmalades and table jellies"
"class_10305","10305","Manufacture of nuts and nut products"
"class_10306","10306","Manufacture of bean curd products"
"class_10401","10401","Manufacture of crude palm oil"
"class_10402","10402","Manufacture of refined palm oil"
"class_10403","10403","Manufacture of palm kernel oil"
"class_10404","10404","Manufacture of crude and refined vegetable oil"
"class_10405","10405","Manufacture of coconut oil"
"class_10406","10406","Manufacture of compound cooking fats"
"class_10407","10407","Manufacture of animal oils and fats"
"class_10501","10501","Manufacture of ice cream and other edible ice such as sorbet"
"class_10502","10502","Manufacture of condensed, powdered and evaporated milk"
"class_10509","10509","Manufacture of other dairy products n.e.c."
"class_10611","10611","Rice milling"
"class_10612","10612","Provision of milling services"
"class_10613","10613","Flour milling"
"class_10619","10619","Manufacture of grain mill products n.e.c."
"class_10621","10621","Manufacture of starches and starch products"
"class_10622","10622","Manufacture of glucose, glucose syrup, maltose, inulin"
"class_10623","10623","Manufacture of sago and tapioca flour/products"
"class_10711","10711","Manufacture of biscuits and cookies"
"class_10712","10712","Manufacture of bread, cakes and other bakery products"
"class_10713","10713","Manufacture of snack products"
"class_10714","10714","Manufacture of frozen bakery products"
"class_10721","10721","Manufacture of sugar"
"class_10722","10722","Manufacture of sugar products"
"class_10731","10731","Manufacture of cocoa products"
"class_10732","10732","Manufacture of chocolate and chocolate products"
"class_10733","10733","Manufacture of sugar confectionery"
"class_10741","10741","Manufacture of meehoon, noodles and other related products"
"class_10742","10742","Manufacture of pastas"
"class_10750","10750","Manufacture of prepared meals and dishes"
"class_10791","10791","Manufacture of coffee"
"class_10792","10792","Manufacture of tea"
"class_10793","10793","Manufacture of sauces and condiments"
"class_10794","10794","Manufacture of spices and curry powder"
"class_10795","10795","Manufacture of egg products"
"class_10799","10799","Manufacture of other food products n.e.c."
"class_10800","10800","Manufacture of prepared animal feeds"
"class_11010","11010","Distilling, rectifying and blending of spirits"
"class_11020","11020","Manufacture of wines"
"class_11030","11030","Manufacture of malt liquors and malt"
"class_11041","11041","Manufacture of soft drinks"
"class_11042","11042","Production of natural mineral water and other bottled water"
"class_12000","12000","MANUFACTURE OF TOBACCO PRODUCTS"
"class_13110","13110","Preparation and spinning of textile fibres"
"class_13120","13120","Weaving of textiles"
"class_13131","13131","Batik making"
"class_13132","13132","Dyeing, bleaching, printing and finishing of yarns and fabrics"
"class_13139","13139","Other finishing textiles"
"class_13910","13910","Manufacture of knitted and crocheted fabrics"
"class_13921","13921","Manufacture of made-up articles of any textile materials, including of knitted or crocheted fabrics"
"class_13922","13922","Manufacture of made-up furnishing articles"
"class_13930","13930","Manufacture of carpets and rugs"
"class_13940","13940","Manufacture of cordage, rope, twine and netting"
"class_13990","13990","Manufacture of other textiles n.e.c."
"class_14101","14101","Manufacture of specific wearing apparel"
"class_14102","14102","Manufacture of clothings"
"class_14103","14103","Custom tailoring"
"class_14109","14109","Manufacture of other clothing accessories"
"class_14200","14200","Manufacture of articles made of fur skins"
"class_14300","14300","Manufacture of knitted and crocheted apparel"
"class_15110","15110","Tanning and dressing of leather; dressing and dyeing of fur"
"class_15120","15120","Manufacture of luggage, handbags and the like, saddlery and harness"
"class_15201","15201","Manufacture of leather footwear"
"class_15202","15202","Manufacture of plastic footwear"
"class_15203","15203","Manufacture of rubber footwear"
"class_15209","15209","Manufacture of other footwear n.e.c."
"class_16211","16211","Manufacture of veneer sheets and plywood"
"class_16221","16221","Manufacture of builders' carpentry"
"class_16222","16222","Manufacture of joinery wood products"
"class_16230","16230","Manufacture of wooden containers"
"class_16291","16291","Manufacture of wood charcoal"
"class_16292","16292","Manufacture of other products of wood, cane, articles of cork, straw and plaiting materials"
"class_17010","17010","Manufacture of pulp, paper and paperboard"
"class_17020","17020","Manufacture of corrugated paper and paperboard and of containers of paper and paperboard"
"class_17091","17091","Manufacture of envelopes and letter-card"
"class_17092","17092","Manufacture of household and personal hygiene paper"
"class_17093","17093","Manufacture of gummed or adhesive paper in strips or rolls and labels and wall paper"
"class_17094","17094","Manufacture of effigies, funeral paper goods, joss paper"
"class_17099","17099","Manufacture of other articles of paper and paperboard n.e.c."
"class_18110","18110","Printing"
"class_18120","18120","Service activities related to printing"
"class_18200","18200","Reproduction of recorded media"
"class_19100","19100","Manufacture of coke oven products"
"class_19201","19201","Manufacture of refined petroleum products"
"class_19202","19202","Manufacture of bio-diesel products"
"class_20111","20111","Manufacture of liquefied or compressed inorganic industrial or medical gases"
"class_20112","20112","Manufacture of basic organic chemicals"
"class_20113","20113","Manufacture of inorganic compounds"
"class_20119","20119","Manufacture of other basic chemicals n.e.c."
"class_20121","20121","Manufacture of fertilizers"
"class_20129","20129","Manufacture of associated nitrogen products"
"class_20131","20131","Manufacture of plastic in primary forms"
"class_20132","20132","Manufacture of synthetic rubber in primary forms: synthetic rubber, factice"
"class_20133","20133","Manufacture of mixtures of synthetic rubber and natural rubber or rubber - like gums"
"class_20210","20210","Manufacture of pesticides and other agrochemical products"
"class_20221","20221","Manufacture of paints, varnishes and similar coatings ink and mastics"
"class_20222","20222","Manufacture of printing ink"
"class_20231","20231","Manufacture of soap and detergents, cleaning and polishing preparations"
"class_20232","20232","Manufacture of perfumes and toilet preparations"
"class_20291","20291","Manufacture of photographic plates, films, sensitized paper and other sensitized unexposed materials"
"class_20292","20292","Manufacture of writing and drawing ink"
"class_20299","20299","Manufacture of other chemical products n.e.c."
"class_20300","20300","Manufacture of man-made fibres"
"class_21001","21001","Manufacture of medicinal active substances to be used for their pharmacological properties in the manufacture of medicaments"
"class_21002","21002","Processing of blood"
"class_21003","21003","Manufacture of medicaments"
"class_21004","21004","Manufacture of chemical contraceptive products"
"class_21005","21005","Manufacture of medical diagnostic preparation"
"class_21006","21006","Manufacture of radioactive in-vivo diagnostic substances"
"class_21007","21007","Manufacture of biotech pharmaceuticals"
"class_21009","21009","Manufacture of other pharmaceuticals, medicinal chemical and botanical products n.e.c."
"class_22111","22111","Manufacture of rubber tyres for vehicles"
"class_22112","22112","Manufacture of interchangeable tyre treads and retreading rubber tyres"
"class_22191","22191","Manufacture of other products of natural or synthetic rubber, unvulcanized, vulcanized or hardened"
"class_22192","22192","Manufacture of rubber gloves"
"class_22193","22193","Rubber remilling and latex processing"
"class_22199","22199","Manufacture of other rubber products n.e.c"
"class_22201","22201","Manufacture of semi-manufactures of plastic products"
"class_22202","22202","Manufacture of finished plastic products"
"class_22203","22203","Manufacture of plastic articles for the packing of goods"
"class_22204","22204","Manufacture of builders' plastics ware"
"class_22205","22205","Manufacture of plastic tableware, kitchenware and toilet articles"
"class_22209","22209","Manufacture of diverse plastic products n.e.c."
"class_23101","23101","Manufacture of flat glass, including wired, coloured or tinted flat glass"
"class_23102","23102","Manufacture of laboratory, hygienic or pharmaceutical glassware"
"class_23109","23109","Manufacture of other glass products n.e.c."
"class_23911","23911","Manufacture of refractory mortars and concretes"
"class_23912","23912","Manufacture of refractory ceramic goods"
"class_23921","23921","Manufacture of non-refractory ceramic"
"class_23929","23929","Manufacture of other clay building materials"
"class_23930","23930","Manufacture of other porcelain and ceramic products"
"class_23941","23941","Manufacture of hydraulic cement"
"class_23942","23942","Manufacture of lime and plaster"
"class_23951","23951","Manufacture of ready-mix and dry-mix concrete and mortars"
"class_23952","23952","Manufacture of precast concrete, cement or artificial stone articles for use in construction"
"class_23953","23953","Manufacture of prefabricated structural components for building or civil engineering of cement, concrete or artificial stone"
"class_23959","23959","Manufacture of other articles of concrete, cement and plaster n.e.c."
"class_23960","23960","Cutting, shaping and finishing of stone"
"class_23990","23990","Manufacture of other non-metallic mineral products n.e.c."
"class_24101","24101","Production of pig iron and spiegeleisen in pigs, blocks or other primary forms"
"class_24102","24102","Production of bars and rods of stainless steel or other alloy steel"
"class_24103","24103","Manufacture of seamless tubes, by hot rolling, hot extrusion or hot drawing, or by cold drawing or cold rolling"
"class_24104","24104","Manufacture of steel tube fittings"
"class_24109","24109","Manufacture of other basic iron and steel products n.e.c."
"class_24201","24201","Tin smelting"
"class_24202","24202","Production of aluminium from alumina"
"class_24209","24209","Manufacture of other basic precious and other non-ferrous metals n.e.c."
"class_24311","24311","Casting of iron"
"class_24312","24312","Casting of steel"
"class_24320","24320","Casting of non-ferrous metals"
"class_25111","25111","Manufacture of industrial frameworks in metal"
"class_25112","25112","Manufacture of prefabricated buildings mainly of metal"
"class_25113","25113","Manufacture of metal doors, windows and their frames, shutters and gates"
"class_25119","25119","Manufacture of other structural metal products"
"class_25120","25120","Manufacture of tanks, reservoirs and containers of metal"
"class_25130","25130","Manufacture of steam generators, except central heating hot water boilers"
"class_25200","25200","Manufacture of weapons and ammunition"
"class_25910","25910","Forging, pressing, stamping and roll-forming of metal; powder metallurgy"
"class_25920","25920","Treatment and coating of metals; machining"
"class_25930","25930","Manufacture of cutlery, hand tools and general hardware"
"class_25991","25991","Manufacture of tins and cans for food products, collapsible tubes and boxes"
"class_25992","25992","Manufacture of metal cable, plaited bands and similar articles"
"class_25993","25993","Manufacture of bolts, screws, nuts and similar threaded products"
"class_25994","25994","Manufacture of metal household articles"
"class_25999","25999","Manufacture of any other fabricated metal products n.e.c."
"class_26101","26101","Manufacture of diodes, transistors and similar semiconductor devices"
"class_26102","26102","Manufacture electronic integrated circuits micro assemblies"
"class_26103","26103","Manufacture of electrical capacitors and resistors"
"class_26104","26104","Manufacture of printed circuit boards"
"class_26105","26105","Manufacture of display components"
"class_26109","26109","Manufacture of other components for electronic applications"
"class_26201","26201","Manufacture of computers"
"class_26202","26202","Manufacture of peripheral equipment"
"class_26300","26300","Manufacture of communication equipment"
"class_26400","26400","Manufacture of consumer electronics"
"class_26511","26511","Manufacture of measuring, testing, navigating and control equipment"
"class_26512","26512","Manufacture of industrial process control equipment"
"class_26520","26520","Manufacture of watches and clocks and parts"
"class_26600","26600","Manufacture of irradiation, electro medical and electrotherapeutic equipment"
"class_26701","26701","Manufacture of optical instruments and equipment"
"class_26702","26702","Manufacture of photographic equipment"
"class_26800","26800","Manufacture of magnetic and optical recording media"
"class_27101","27101","Manufacture of electric motors, generators and transformers"
"class_27102","27102","Manufacture of electricity distribution and control apparatus"
"class_27200","27200","Manufacture of batteries and accumulators"
"class_27310","27310","Manufacture of fibre optic cables"
"class_27320","27320","Manufacture of other electronic and electric wires and cables"
"class_27330","27330","Manufacture of current-carrying and non current-carrying wiring devices for electrical circuits regardless of material"
"class_27400","27400","Manufacture of electric lighting equipment"
"class_27500","27500","Manufacture of domestic appliances"
"class_27900","27900","Manufacture of miscellaneous electrical equipment other than motors, generators and transformers, batteries and accumulators, wires and wiring devices, lighting equipment or domestic appliances"
"class_28110","28110","Manufacture of engines and turbines, except aircraft, vehicle and cycle engines"
"class_28120","28120","Manufacture of fluid power equipment"
"class_28130","28130","Manufacture of other pumps, compressors, taps and valves"
"class_28140","28140","Manufacture of bearings, gears, gearing and driving elements"
"class_28150","28150","Manufacture of ovens, furnaces and furnace burners"
"class_28160","28160","Manufacture of lifting and handling equipment"
"class_28170","28170","Manufacture of office machinery and equipment (except computers and peripheral equipment)"
"class_28180","28180","Manufacture of power-driven hand tools with self-contained electric or non-electric motor or pneumatic drives"
"class_28191","28191","Manufacture of refrigerating or freezing industrial equipment"
"class_28192","28192","Manufacture of air-conditioning machines, including for motor vehicles"
"class_28199","28199","Manufacture of other general-purpose machinery n.e.c."
"class_28210","28210","Manufacture of agricultural and forestry machinery"
"class_28220","28220","Manufacture of metal-forming machinery and machine tools"
"class_28230","28230","Manufacture of machinery for metallurgy"
"class_28240","28240","Manufacture of machinery for mining, quarrying and construction"
"class_28250","28250","Manufacture of machinery for food, beverage and tobacco processing"
"class_28260","28260","Manufacture of machinery for textile, apparel and leather production"
"class_28290","28290","Manufacture of other special-purpose machinery n.e.c."
"class_29101","29101","Manufacture of passenger cars"
"class_29102","29102","Manufacture of commercial vehicles"
"class_29200","29200","Manufacture of bodies (coachwork) for motor vehicles; manufacture of trailers and semi- trailers"
"class_29300","29300","Manufacture of parts and accessories for motor vehicles"
"class_30110","30110","Building of ships and floating structures"
"class_30120","30120","Building of pleasure and sporting boats"
"class_30200","30200","Manufacture of railway locomotives and rolling stock"
"class_30300","30300","Manufacture of air and spacecraft and related machinery"
"class_30400","30400","Manufacture of military fighting vehicles"
"class_30910","30910","Manufacture of motorcycles"
"class_30920","30920","Manufacture of bicycles and invalid carriages"
"class_30990","30990","Manufacture of other transport equipments n.e.c."
"class_31001","31001","Manufacture of wooden and cane furniture"
"class_31002","31002","Manufacture of metal furniture"
"class_31003","31003","Manufacture of mattress"
"class_31009","31009","Manufacture of other furniture, except of stone, concrete or ceramic"
"class_32110","32110","Manufacture of jewellery and related articles"
"class_32120","32120","Manufacture of imitation jewellery and related articles"
"class_32200","32200","Manufacture of musical instruments"
"class_32300","32300","Manufacture of sports goods"
"class_32400","32400","Manufacture of games and toys"
"class_32500","32500","Manufacture of medical and dental instrument and supplies"
"class_32901","32901","Manufacture of stationery"
"class_32909","32909","Other manufacturing n.e.c."
"class_33110","33110","Repair of fabricated metal products"
"class_33120","33120","Repair and maintenance of industrial machinery and equipment"
"class_33131","33131","Repair and maintenance of the measuring, testing, navigating and control equipment"
"class_33132","33132","Repair and maintenance of irradiation, electro medical and electrotherapeutic equipment"
"class_33133","33133","Repair of optical instruments and photographic equipment"
"class_33140","33140","Repair and maintenance of electrical equipment except domestic appliances"
"class_33150","33150","Repair and maintenance of transport equipment except motorcycles and bicycles"
"class_33190","33190","Repair and maintenance of other equipment n.e.c."
"class_33200","33200","Installation of industrial machinery and equipment"
"class_35101","35101","Operation of generation facilities that produce electric energy"
"class_35102","35102","Operation of transmission, distribution and sales of electricity"
"class_35201","35201","Manufacture of gaseous fuels with a specified calorific value, by purification, blending and other processes from gases of various types including natural gas"
"class_35202","35202","Transportation, distribution and supply of gaseous fuels of all kinds through a system of mains"
"class_35203","35203","Sale of gas to the user through mains"
"class_35301","35301","Production, collection and distribution of steam and hot water for heating, power and other purposes"
"class_35302","35302","Production and distribution of cooled air, chilled water for cooling purposes"
"class_35303","35303","Production of ice, including ice for food and non-food (e.g. cooling) purposes"
"class_36001","36001","Purification and distribution of water for water supply purposes"
"class_36002","36002","Desalting of sea or ground water to produce water as the principal product of interest"
"class_37000","37000","Sewerage and similar activities"
"class_38111","38111","Collection of non-hazardous solid waste (i.e. garbage) within a local area"
"class_38112","38112","Collection of recyclable materials"
"class_38113","38113","Collection of refuse in litter-bins in public places"
"class_38114","38114","Collection of construction and demolition waste"
"class_38115","38115","Operation of waste transfer stations for non-hazardous waste"
"class_38121","38121","Collection of hazardous waste"
"class_38122","38122","Operation of waste transfer stations for hazardous waste"
"class_38210","38210","Treatment and disposal of non-hazardous waste"
"class_38220","38220","Treatment and disposal of hazardous waste"
"class_38301","38301","Mechanical crushing of metal waste"
"class_38302","38302","Dismantling of automobiles, computers, televisions and other equipment for material recover"
"class_38303","38303","Reclaiming of rubber such as used tires to produce secondary raw material"
"class_38304","38304","Reuse of rubber products"
"class_38309","38309","Materials recovery n.e.c."
"class_39000","39000","Remediation activities and other waste management services"
"class_41001","41001","Residential buildings"
"class_41002","41002","Non-residential buildings"
"class_41003","41003","Assembly and erection of prefabricated constructions on the site"
"class_41009","41009","Construction of buildings n.e.c."
"class_42101","42101","Construction of motorways, streets, roads, other vehicular and pedestrian ways"
"class_42102","42102","Surface work on streets, roads, highways, bridges or tunnels"
"class_42103","42103","Construction of bridges, including those for elevated highways"
"class_42104","42104","Construction of tunnels"
"class_42105","42105","Construction of railways and subways"
"class_42106","42106","Construction of airfield/airports runways"
"class_42109","42109","Construction of roads and railways n.e.c."
"class_42201","42201","Long-distance pipelines, communication and power lines"
"class_42202","42202","Urban pipelines, urban communication and power lines; ancillary urban works"
"class_42203","42203","Water main and line construction"
"class_42204","42204","Reservoirs"
"class_42205","42205","Construction of irrigation systems (canals)"
"class_42206","42206","Construction of sewer systems (including repair) and sewage disposal plants"
"class_42207","42207","Construction of power plants"
"class_42209","42209","Construction of utility projects n.e.c."
"class_42901","42901","Construction of refineries"
"class_42902","42902","Construction of waterways, harbour and river works, pleasure ports (marinas), locks"
"class_42903","42903","Construction of dams and dykes"
"class_42904","42904","Dredging of waterways"
"class_42905","42905","Outdoor sports facilities"
"class_42906","42906","Land subdivision with land improvement"
"class_42909","42909","Construction of other engineering projects n.e.c."
"class_43110","43110","Demolition or wrecking of buildings and other structures"
"class_43121","43121","Clearing of building sites"
"class_43122","43122","Earth moving"
"class_43123","43123","Drilling, boring and core sampling for construction, geophysical, geological or similar purposes"
"class_43124","43124","Site preparation for mining"
"class_43125","43125","Drainage of agricultural or forestry land"
"class_43126","43126","Land reclamation work"
"class_43129","43129","Other site preparation activities n.e.c."
"class_43211","43211","Electrical wiring and fittings"
"class_43212","43212","Telecommunications wiring"
"class_43213","43213","Computer network and cable television wiring"
"class_43214","43214","Satellite dishes"
"class_43215","43215","Lighting systems"
"class_43216","43216","Security systems"
"class_43219","43219","Electrical installation n.e.c."
"class_43221","43221","Installation of heating systems (electric, gas and oil)"
"class_43222","43222","Installation of furnaces, cooling towers"
"class_43223","43223","Installation of non-electric solar energy collectors"
"class_43224","43224","Installation of plumbing and sanitary equipment"
"class_43225","43225","Installation of ventilation, refrigeration or air-conditioning equipment and ducts"
"class_43226","43226","Installation of gas fittings"
"class_43227","43227","Installation of fire and lawn sprinkler systems"
"class_43228","43228","Steam piping"
"class_43229","43229","Plumbing, heat and air-conditioning installation n.e.c."
"class_43291","43291","Installation of elevators, escalators in buildings or other construction projects"
"class_43292","43292","Installation of automated and revolving doors in buildings or other construction projects"
"class_43293","43293","Installation of lighting conductors in buildings or other construction projects"
"class_43294","43294","Installation vacuum cleaning systems in buildings or other construction projects"
"class_43295","43295","Installation thermal, sound or vibration insulation in buildings or other construction projects"
"class_43299","43299","Other construction installation n.e.c."
"class_43301","43301","Installation of doors, windows, door and window frames of wood or other materials, fitted kitchens, staircases, shop fittings and furniture"
"class_43302","43302","Laying, tiling, hanging or fitting in buildings or other construction projects of various types of materials"
"class_43303","43303","Interior and exterior painting of buildings"
"class_43304","43304","Painting of civil engineering structures"
"class_43305","43305","Installation of glass, mirrors"
"class_43306","43306","Interior completion"
"class_43307","43307","Cleaning of new buildings after construction"
"class_43309","43309","Other building completion and finishing work n.e.c."
"class_43901","43901","Construction of foundations, including pile driving"
"class_43902","43902","Erection of non-self-manufactured steel elements"
"class_43903","43903","Scaffolds and work platform erecting and dismantling"
"class_43904","43904","Bricklaying and stone setting"
"class_43905","43905","Construction of outdoor swimming pools"
"class_43906","43906","Steam cleaning, sand blasting and similar activities for building exteriors"
"class_43907","43907","Renting of construction machinery and equipment with operator (e.g. cranes)"
"class_43909","43909","Other specialized construction activities, n.e.c."
"class_45101","45101","Wholesale and retail of new motor vehicles"
"class_45102","45102","Wholesale and retail of used motor vehicles"
"class_45103","45103","Sale of industrial, commercial and agriculture vehicles – new"
"class_45104","45104","Sale of industrial, commercial and agriculture vehicles – used"
"class_45105","45105","Sale by commission agents"
"class_45106","45106","Car auctions"
"class_45109","45109","Sale of other motor vehicles n.e.c."
"class_45201","45201","Maintenance and repair of motor vehicles"
"class_45202","45202","Spraying and painting"
"class_45203","45203","Washing and polishing (car wash)"
"class_45204","45204","Repair of motor vehicle seats"
"class_45205","45205","Installation of parts and accessories not as part of the manufacturing process"
"class_45300","45300","Wholesale and retail sale of all kinds of parts, components, supplies, tools and accessories for motor vehicles"
"class_45401","45401","Wholesale and retail sale of motorcycles"
"class_45402","45402","Wholesale and retail sale of parts and accessories for motorcycles"
"class_45403","45403","Repair and maintenance of motorcycles"
"class_46100","46100","Wholesale on a fee or contract basis"
"class_46201","46201","Wholesale of rubber"
"class_46202","46202","Wholesale of palm oil"
"class_46203","46203","Wholesale of lumber and timber"
"class_46204","46204","Wholesale of flowers and plants"
"class_46205","46205","Wholesale of livestock"
"class_46209","46209","Wholesale of agricultural raw material and live animal n.e.c."
"class_46311","46311","Wholesale of meat, poultry and eggs"
"class_46312","46312","Wholesale of fish and other seafood"
"class_46313","46313","Wholesale of fruits"
"class_46314","46314","Wholesale of vegetables"
"class_46319","46319","Wholesale of meat, fish, fruits and vegetables n.e.c."
"class_46321","46321","Wholesale of rice, other grains, flour and sugars"
"class_46322","46322","Wholesale of dairy products"
"class_46323","46323","Wholesale of confectionary"
"class_46324","46324","Wholesale of  biscuits, cakes, breads and other bakery products"
"class_46325","46325","Wholesale of coffee, tea, cocoa and other beverages"
"class_46326","46326","Wholesale of beer, wine and spirits"
"class_46327","46327","Wholesale of tobacco, cigar, cigarettes"
"class_46329","46329","Wholesale of other foodstuffs"
"class_46411","46411","Wholesale of yarn and fabrics"
"class_46412","46412","Wholesale of household linen, towels, blankets"
"class_46413","46413","Wholesale of clothing"
"class_46414","46414","Wholesale of clothing accessories"
"class_46415","46415","Wholesale of fur articles"
"class_46416","46416","Wholesale of footwear"
"class_46417","46417","Wholesale of haberdashery"
"class_46419","46419","Wholesale of textiles, clothing n.e.c."
"class_46421","46421","Wholesale of pharmaceutical and medical goods"
"class_46422","46422","Wholesale of perfumeries, cosmetics, soap and toiletries"
"class_46431","46431","Wholesale of bicycles and their parts and accessories"
"class_46432","46432","Wholesale of photographic and optical goods"
"class_46433","46433","Wholesale of leather goods and travel accessories"
"class_46434","46434","Wholesale of musical instruments, games and toys, sports goods"
"class_46441","46441","Wholesale of handicrafts and artificial flowers"
"class_46442","46442","Wholesale of cut flowers and plants"
"class_46443","46443","Wholesale of watches and clocks"
"class_46444","46444","Wholesale of jewellery"
"class_46491","46491","Wholesale of household furniture"
"class_46492","46492","Wholesale of household appliances"
"class_46493","46493","Wholesale of lighting equipment"
"class_46494","46494","Wholesale of household utensils and cutlery, crockery, glassware, chinaware and pottery"
"class_46495","46495","Wholesale of woodenware, wickerwork and corkware"
"class_46496","46496","Wholesale of electrical and electronic goods"
"class_46497","46497","Wholesale of stationery, books, magazines and newspapers"
"class_46499","46499","Wholesale of other household goods n.e.c."
"class_46510","46510","Wholesale of computer hardware, software and peripherals"
"class_46521","46521","Wholesale of telephone and telecommunications equipment, cell phones, pagers"
"class_46522","46522","Wholesale of electronic components and wiring accessories"
"class_46531","46531","Wholesale of agricultural machinery, equipment and supplies"
"class_46532","46532","Wholesale of lawn mowers however operated"
"class_46591","46591","Wholesale of office machinery and business equipment, except computers and computer peripheral equipment"
"class_46592","46592","Wholesale of office furniture"
"class_46593","46593","Wholesale of computer-controlled machines tools"
"class_46594","46594","Wholesale of industrial machinery, equipment and supplies"
"class_46595","46595","Wholesale of construction and civil engineering machinery and equipment"
"class_46596","46596","Wholesale of lift escalators, air-conditioning, security and fire fighting equipment"
"class_46599","46599","Wholesale of other machinery for use in industry, trade and navigation and other services n.e.c."
"class_46611","46611","Wholesale of petrol, diesel, lubricants"
"class_46612","46612","Wholesale of liquefied petroleum gas"
"class_46619","46619","Wholesale of other solid, liquid and gaseous fuels and related products n.e.c."
"class_46621","46621","Wholesale of ferrous and non-ferrous metal ores and metals"
"class_46622","46622","Wholesale of ferrous and non-ferrous semi-finished metal ores and products n.e.c."
"class_46631","46631","Wholesale of logs, sawn timber, plywood, veneer and related products"
"class_46632","46632","Wholesale of paints and varnish"
"class_46633","46633","Wholesale of construction materials"
"class_46634","46634","Wholesale of fittings and fixtures"
"class_46635","46635","Wholesale of hot water heaters"
"class_46636","46636","Wholesale of sanitary installation and equipment"
"class_46637","46637","Wholesale of tools"
"class_46639","46639","Wholesale of other construction materials, hardware, plumbing and heating equipment and supplies n.e.c."
"class_46691","46691","Wholesale of industrial chemicals"
"class_46692","46692","Wholesale of fertilizers and agrochemical products"
"class_46693","46693","Wholesale of plastic materials in primary forms"
"class_46694","46694","Wholesale of rubber scrap"
"class_46695","46695","Wholesale of textile fibres"
"class_46696","46696","Wholesale of paper in bulk, packaging materials"
"class_46697","46697","Wholesale of precious stones"
"class_46698","46698","Wholesale of metal and non-metal waste and scrap and materials for recycling"
"class_46699","46699","Dismantling of automobiles, computer, televisions and other equipment to obtain and re-sell usable parts"
"class_46901","46901","Wholesale of aquarium fishes, pet birds and animals"
"class_46902","46902","Wholesale of animal/pet food"
"class_46909","46909","Wholesale of a variety of goods without any particular specialization n.e.c."
"class_47111","47111","Provision stores"
"class_47112","47112","Supermarket"
"class_47113","47113","Mini market"
"class_47114","47114","Convenience stores"
"class_47191","47191","Department stores"
"class_47192","47192","Department stores and supermarket"
"class_47193","47193","Hypermarket"
"class_47194","47194","News agent and miscellaneous goods store"
"class_47199","47199","Other retail sale in non-specialized stores n.e.c."
"class_47211","47211","Retail sale of rice, flour, other grains and sugars"
"class_47212","47212","Retail sale of fresh or preserved vegetables and fruits"
"class_47213","47213","Retail sale of dairy products and eggs"
"class_47214","47214","Retail sale of meat and meat products (including poultry)"
"class_47215","47215","Retail sale of fish, other seafood and products thereof"
"class_47216","47216","Retail sale of bakery products and sugar confectionery"
"class_47217","47217","Retail sale of mee, kuey teow, mee hoon, wantan skins and other food products made from flour or soya"
"class_47219","47219","Retail sale of other food products n.e.c."
"class_47221","47221","Retail sale of beer, wine and spirits"
"class_47222","47222","Retail sale of tea, coffee, soft drinks, mineral water and other beverages"
"class_47230","47230","Retail sale of tobacco products in specialized store"
"class_47300","47300","Retail sale of automotive fuel in specialized stores"
"class_47412","47412","Retail sale of video game consoles and non-customized software"
"class_47413","47413","Retail sale of telecommunication equipment"
"class_47420","47420","Retail sale of audio and video equipment in specialized store"
"class_47510","47510","Retail sale of textiles in specialized stores"
"class_47531","47531","Retail sale of carpets and rugs"
"class_47532","47532","Retail sale of curtains and net curtains"
"class_47533","47533","Retail sale of wallpaper and floor coverings"
"class_47591","47591","Retail sale of household furniture"
"class_47592","47592","Retail sale of articles for lighting"
"class_47593","47593","Retail sale of household utensils and cutlery, crockery, glassware, chinaware and pottery"
"class_47594","47594","Retail sale of wood, cork goods and wickerwork goods"
"class_47595","47595","Retail sale of household appliances"
"class_47596","47596","Retail sale of musical instruments and scores"
"class_47597","47597","Retail sale of security systems"
"class_47598","47598","Retail sale of household articles and equipment n.e.c."
"class_47611","47611","Retail sale of office supplies and equipment"
"class_47612","47612","Retail sale of books, newspapers and stationary"
"class_47631","47631","Retail sale of sports goods and equipments"
"class_47632","47632","Retail sale of fishing equipment"
"class_47633","47633","Retail sale of camping goods"
"class_47634","47634","Retail sale of boats and equipments"
"class_47635","47635","Retail sale of bicycles and related parts and accessories"
"class_47640","47640","Retail sale of games and toys, made of all materials"
"class_47711","47711","Retail sale of articles of clothing, articles of fur and clothing accessories"
"class_47712","47712","Retail sale of footwear"
"class_47713","47713","Retail sale of leather goods, accessories of leather and leather substitutes"
"class_47721","47721","Stores specialized in retail sale of pharmaceuticals, medical and orthopaedic goods"
"class_47722","47722","Stores specialized in retail sale of perfumery, cosmetic and toilet articles"
"class_47731","47731","Retail sale of photographic and precision equipment"
"class_47732","47732","Retail sale of watches and clocks"
"class_47733","47733","Retail sale of jewellery"
"class_47734","47734","Retail sale of flowers, plants, seeds, fertilizers"
"class_47735","47735","Retail sale of souvenirs, craftwork and religious articles"
"class_47736","47736","Retail sale of household fuel oil, cooking gas, coal and fuel wood"
"class_47737","47737","Retail sale of spectacles and other optical goods"
"class_47738","47738","Retail sale of aquarium fishes, pet animals and pet food"
"class_47739","47739","Other retail sale of new goods in specialized stores n.e.c."
"class_47741","47741","Retail sale of second-hand books"
"class_47742","47742","Retail sale of second-hand electrical and electronic goods"
"class_47743","47743","Retail sale of antiques"
"class_47744","47744","Activities of auctioning houses (retail)"
"class_47749","47749","Retail sale of second-hand goods n.e.c."
"class_47810","47810","Retail sale of food, beverages and tobacco products via stalls or markets"
"class_47820","47820","Retail sale of textiles, clothing and footwear via stalls or markets"
"class_47891","47891","Retail sale of carpets and rugs via stalls or markets"
"class_47893","47893","Retail sale of games and toys via stalls or markets"
"class_47894","47894","Retail sale of household appliances and consumer electronics via stall or markets"
"class_47895","47895","Retail sale of music and video recordings via stall or markets"
"class_47911","47911","Retail sale of any kind of product by mail order"
"class_47912","47912","Retail sale of any kind of product over the Internet"
"class_47913","47913","Direct sale via television, radio and telephone"
"class_47914","47914","Internet retail auctions"
"class_47992","47992","Retail sale of any kind of product through vending machines"
"class_47999","47999","Other retail sale not in stores, stalls or markets n.e.c."
"class_49110","49110","Passenger transport by inter-urban railways"
"class_49120","49120","Freight transport by inter-urban, suburban and urban railways"
"class_49211","49211","City bus services"
"class_49212","49212","Urban and suburban railway passenger transport service"
"class_49221","49221","Express bus services"
"class_49222","49222","Employees bus services"
"class_49223","49223","School bus services"
"class_49224","49224","Taxi operation and limousine services"
"class_49225","49225","Rental of cars with driver"
"class_49229","49229","Other passenger land transport n.e.c."
"class_49230","49230","Freight transport by road"
"class_49300","49300","Transport via pipeline"
"class_50111","50111","Operation of excursion, cruise or sightseeing boats"
"class_50112","50112","Operation of ferries, water taxis"
"class_50113","50113","Rental of pleasure boats with crew for sea and coastal water transport"
"class_50121","50121","Transport of freight overseas and coastal waters, whether scheduled or not"
"class_50122","50122","Transport by towing or pushing of barges, oil rigs"
"class_50211","50211","Transport of passenger via rivers, canals, lakes and other inland waterways"
"class_50212","50212","Rental of pleasure boats with crew for inland water transport"
"class_50220","50220","Transport of freight via rivers, canals, lakes and other inland waterways"
"class_51101","51101","Transport of passengers by air over regular routes and on regular schedules"
"class_51102","51102","Non-scheduled transport of passenger by air"
"class_51103","51103","Renting of air-transport equipment with operator for the purpose of passenger transportation"
"class_51201","51201","Transport freight by air over regular routes and on regular schedules"
"class_51202","51202","Non-scheduled transport of freight by air"
"class_51203","51203","Renting of air-transport equipment with operator for the purpose of freight transportation"
"class_52100","52100","Warehousing and storage services"
"class_52211","52211","Operation of terminal facilities"
"class_52212","52212","Towing and road side assistance"
"class_52213","52213","Operation of parking facilities for motor vehicles (parking lots)"
"class_52214","52214","Highway, bridge and tunnel operation services"
"class_52219","52219","Other service activities incidental to land transportation n.e.c."
"class_52221","52221","Port, harbours and piers operation services"
"class_52222","52222","Vessel salvage and refloating services"
"class_52229","52229","Other service activities incidental to water transportation n.e.c."
"class_52231","52231","Operation of terminal facilities"
"class_52232","52232","Airport and air-traffic-control activities"
"class_52233","52233","Ground service activities on airfields"
"class_52234","52234","Fire fighting and fire-prevention services at airports"
"class_52239","52239","Other service activities incidental to air transportation n.e.c."
"class_52241","52241","Stevedoring services"
"class_52249","52249","Other cargo handling activities n.e.c."
"class_52291","52291","Forwarding of freight"
"class_52292","52292","Brokerage for ship and aircraft space"
"class_52299","52299","Other transportation support activities n.e.c."
"class_53100","53100","National postal services"
"class_53200","53200","Courier activities other than national post activities"
"class_55101","55101","Hotels and resort hotels"
"class_55102","55102","Motels"
"class_55103","55103","Apartment hotels"
"class_55104","55104","Chalets"
"class_55105","55105","Rest house/guest house"
"class_55106","55106","Bed and breakfast units"
"class_55107","55107","Hostels"
"class_55108","55108","Home stay"
"class_55109","55109","Other short term accommodation activities n.e.c."
"class_55200","55200","Camping grounds, recreational vehicle parks and trailer parks"
"class_55900","55900","Other accommodation"
"class_56103","56103","Fast-food restaurants"
"class_56104","56104","Ice cream truck vendors and parlours"
"class_56105","56105","Mobile food carts"
"class_56106","56106","Food stalls/hawkers"
"class_56107","56107","Food or beverage, food and beverage preparation in market stalls/hawkers"
"class_56210","56210","Event/food caterers"
"class_56290","56290","Other food service activities"
"class_56301","56301","Pubs, bars, discotheques, coffee houses, cocktail lounges and karaoke"
"class_56302","56302","Coffee shops"
"class_56303","56303","Drink stalls/hawkers"
"class_56304","56304","Mobile beverage"
"class_56309","56309","Others drinking places n.e.c."
"class_58110","58110","Publishing of books, brochures and other publications"
"class_58120","58120","Publishing of mailing lists, telephone book, other directories"
"class_58130","58130","Publishing of newspapers, journals, magazines and periodicals in print or electronic form"
"class_58190","58190","Publishing of catalogues, photos, engraving and postcards, greeting cards, forms, posters, reproduction of works of art, advertising material and other printed matter n.e.c."
"class_58201","58201","Business and other applications"
"class_58202","58202","Computer games for all platforms"
"class_58203","58203","Operating systems"
"class_59110","59110","Motion picture, video and television programme production activities"
"class_59120","59120","Motion picture, video and television programme post-production activities"
"class_59130","59130","Motion picture, video and television programme distribution activities"
"class_59140","59140","Motion picture projection activities"
"class_59200","59200","Sound recording and music publishing activities"
"class_60100","60100","Radio broadcasting"
"class_60200","60200","Television programming and broadcasting activities"
"class_61101","61101","Wired telecommunications services"
"class_61102","61102","Internet access providers by the operator of the wired infrastructure"
"class_61201","61201","Wireless telecommunications services"
"class_61202","61202","Internet access providers by the operator of the wireless infrastructure"
"class_61300","61300","Satellite telecommunications services"
"class_61901","61901","Provision of Internet access over networks between the client and the ISP not owned or controlled by the ISP"
"class_61902","61902","Provision of telecommunications services over existing telecom connection"
"class_61903","61903","Telecommunications resellers"
"class_61904","61904","Provision of telecommunications services over existing telecom connections VOIP (Voice Over Internet Protocol) provision"
"class_61905","61905","Provision of specialized telecommunications applications"
"class_61909","61909","Other telecommunications activities n.e.c."
"class_62010","62010","Computer programming activities"
"class_62021","62021","Computer consultancy"
"class_62022","62022","Computer facilities management activities"
"class_62091","62091","Information Communication Technology (ICT) system security"
"class_62099","62099","Other information technology service activities n.e.c."
"class_63111","63111","Activities of providing infrastructure for hosting, data processing services and related activities"
"class_63112","63112","Data processing activities"
"class_63120","63120","Web portals"
"class_63910","63910","News syndicate and news agency activities"
"class_63990","63990","Other information service activities n.e.c."
"class_64110","64110","Central banking"
"class_64191","64191","Commercial Banks"
"class_64192","64192","Islamic Banks"
"class_64193","64193","Offshore Banks"
"class_64194","64194","Investment Banks"
"class_64195","64195","Development financial institutions (with deposit taking functions)"
"class_64199","64199","Other monetary intermediation (with deposit taking functions) n.e.c."
"class_64200","64200","Activities of holding companies"
"class_64301","64301","Venture capital companies"
"class_64302","64302","Unit trust fund excludes REITs"
"class_64303","64303","Property unit trust (REITs)"
"class_64304","64304","Other administration of trusts accounts"
"class_64309","64309","Trusts, funds and similar financial entities n.e.c."
"class_64910","64910","Financial leasing activities"
"class_64921","64921","Development financial institutions (without deposit taking functions)"
"class_64922","64922","Credit card services"
"class_64923","64923","Licensed money lending activities"
"class_64924","64924","Pawnshops and pawnbrokers includes Ar-Rahnu"
"class_64925","64925","Co-operative with credits functions"
"class_64929","64929","Other credit granting n.e.c."
"class_64991","64991","Factoring companies"
"class_64992","64992","Representative office of foreign banks"
"class_64993","64993","Nominee companies"
"class_64999","64999","Other financial service activities, except insurance/takaful and pension funding n.e.c."
"class_65111","65111","Life insurance"
"class_65112","65112","Family takaful"
"class_65121","65121","General insurance"
"class_65122","65122","General takaful"
"class_65123","65123","Composite insurance"
"class_65124","65124","Offshore insurance"
"class_65125","65125","Offshore takaful"
"class_65201","65201","Life reinsurance"
"class_65202","65202","Family retakaful"
"class_65203","65203","General reinsurance"
"class_65204","65204","General retakaful"
"class_65205","65205","Composite retakaful"
"class_65206","65206","Offshore reinsurance"
"class_65207","65207","Offshore retakaful"
"class_65301","65301","Pension funding"
"class_65302","65302","Provident funding"
"class_66111","66111","Stock exchanges"
"class_66112","66112","Exchanges for commodity contracts"
"class_66113","66113","Securities exchange"
"class_66114","66114","Exchanges for commodity futures contracts"
"class_66119","66119","Administration of financial markets n.e.c."
"class_66121","66121","Stock, share and bond brokers"
"class_66122","66122","Commodity brokers and dealers"
"class_66123","66123","Gold bullion dealers"
"class_66124","66124","Foreign exchange broker and dealers (Bureaux de change)"
"class_66125","66125","Money-changing services"
"class_66129","66129","Other financial and commodity futures brokers and dealers"
"class_66191","66191","Investment advisory services"
"class_66192","66192","Financial consultancy services"
"class_66199","66199","Activities auxiliary to finance n.e.c."
"class_66211","66211","Insurance adjusting service"
"class_66212","66212","Takaful adjusting service"
"class_66221","66221","Insurance agents"
"class_66222","66222","Takaful agents"
"class_66223","66223","Insurance brokers"
"class_66224","66224","Takaful brokers"
"class_66290","66290","Other activities auxiliary to insurance, takaful and pension funding"
"class_66301","66301","Management of pension funds"
"class_66302","66302","Assets/portfolio management"
"class_66303","66303","Unit trust management companies"
"class_68101","68101","Buying, selling, renting and operating of self-owned or leased real estate – residential buildings"
"class_68102","68102","Buying, selling, renting and operating of self-owned or leased real estate – non-residential buildings"
"class_68103","68103","Buying, selling, renting and operating of self-owned or leased real estate – land"
"class_68104","68104","Development of building projects for own operation, i.e. for renting of space in these buildings"
"class_68109","68109","Real estate activities with own or leased property n.e.c."
"class_68201","68201","Activities of real estate agents and brokers for buying, selling and renting of real estate"
"class_68202","68202","Management of real estate on a fee or contract basis"
"class_68203","68203","Appraisal services for real estate"
"class_68209","68209","Real estate activities on a fee or contract basis n.e.c."
"class_69100","69100","Legal activities"
"class_69200","69200","Accounting, bookkeeping and auditing activities; tax consultancy"
"class_70100","70100","Activities of head offices"
"class_70201","70201","Business management consultancy services"
"class_70202","70202","Human resource consultancy services"
"class_70203","70203","Consultancy services in public relation and communications"
"class_70209","70209","Other management consultancy activities n.e.c"
"class_71101","71101","Architectural services"
"class_71102","71102","Engineering services"
"class_71103","71103","Land surveying services"
"class_71109","71109","Other architectural and engineering activities and related technical consultancy n.e.c."
"class_71200","71200","Technical testing and analysis"
"class_72101","72101","Research and development on natural sciences"
"class_72102","72102","Research and development on engineering and technology"
"class_72103","72103","Research and development on medical sciences"
"class_72104","72104","Research and development on biotechnology"
"class_72105","72105","Research and development on agricultural sciences"
"class_72106","72106","Research and development on Information Communication Technology (ICT)"
"class_72109","72109","Research and development on other natural science and engineering n.e.c."
"class_72201","72201","Research and development on social sciences"
"class_72202","72202","Research and development on humanities"
"class_72209","72209","Research and development of other social sciences and humanities n.e.c."
"class_73100","73100","Advertising"
"class_73200","73200","Market research and public opinion polling"
"class_74101","74101","Activities of interior decorators"
"class_74102","74102","Services of graphic designers"
"class_74103","74103","Fashion design services"
"class_74109","74109","Specialized design activities n.e.c."
"class_74200","74200","Photographic activities"
"class_74901","74901","Translation and interpretation activities"
"class_74902","74902","Business brokerage activities"
"class_74903","74903","Security consulting"
"class_74904","74904","Activities of quantity surveyors"
"class_74905","74905","Activities of consultants other than architecture, engineering and management consultants"
"class_74909","74909","Any other professional, scientific and technical activities n.e.c."
"class_75000","75000","VETERINARY ACTIVITIES"
"class_77101","77101","Renting and operational leasing of passenger cars (without driver)"
"class_77102","77102","Renting and operational leasing of trucks, utility trailers and recreational vehicles"
"class_77211","77211","Renting and leasing of pleasure boats, canoes, sailboats"
"class_77212","77212","Renting and leasing of bicycles"
"class_77213","77213","Renting and leasing of beach chairs and umbrellas"
"class_77219","77219","Renting and leasing of other sports equipment n.e.c."
"class_77220","77220","Renting of video tapes, records, CDs, DVDs"
"class_77291","77291","Renting and leasing of textiles, wearing apparel and footwear"
"class_77292","77292","Renting and leasing of furniture, pottery and glass, kitchen and tableware, electrical appliances and house wares"
"class_77293","77293","Renting and leasing of jewellery, musical instruments, scenery and costumes"
"class_77294","77294","Renting and leasing of books, journals and magazines"
"class_77295","77295","Renting and leasing of machinery and equipment used by amateurs or as a hobby"
"class_77296","77296","Renting of flowers and plants"
"class_77297","77297","Renting and leasing of electronic equipment for household use"
"class_77299","77299","Renting and leasing of other personal and household goods n.e.c."
"class_77301","77301","Renting and operational leasing, without operator, of other machinery and equipment that are generally used as capital goods by industries"
"class_77302","77302","Renting and operational leasing of land-transport equipment (other than motor vehicles) without drivers"
"class_77303","77303","Renting and operational leasing of water-transport equipment without operator"
"class_77304","77304","Renting and operational leasing of air transport equipment without operator"
"class_77305","77305","Renting and operational leasing of agricultural and forestry machinery and equipment without operator"
"class_77306","77306","Renting and operational leasing of construction and civil-engineering machinery and equipment without operator"
"class_77307","77307","Rental and operational leasing of office machinery and equipment without operator"
"class_77309","77309","Renting and leasing of other machinery, equipment and tangible goods n.e.c."
"class_77400","77400","Leasing of intellectual property and similar products, except copyrighted works"
"class_78100","78100","Activities of employment placement agencies"
"class_78200","78200","Temporary employment agency activities"
"class_78300","78300","Provision of human resources for client businesses"
"class_79110","79110","Travel agency activities"
"class_79120","79120","Tour operator activities"
"class_79900","79900","Other reservation service and related activities"
"class_80100","80100","Private security activities"
"class_80200","80200","Security systems service activities"
"class_80300","80300","Investigation and detective activities"
"class_81100","81100","Combined facilities support activities"
"class_81210","81210","General cleaning of buildings"
"class_81291","81291","Cleaning of buildings of all types"
"class_81292","81292","Swimming pool cleaning and maintenance services"
"class_81293","81293","Cleaning of industrial machinery"
"class_81294","81294","Cleaning of trains, buses, planes"
"class_81295","81295","Cleaning of pest control services not in connection with agriculture"
"class_81296","81296","Disinfecting and exterminating activities"
"class_81297","81297","Cleaning of sea tankers"
"class_81299","81299","Other building and industrial cleaning activities, n.e.c."
"class_81300","81300","Landscape care and maintenance service activities"
"class_82110","82110","Combined office administrative service activities"
"class_82191","82191","Document preparation, editing and/or proofreading"
"class_82192","82192","Typing, word processing or desktop publishing"
"class_82193","82193","Secretarial support services"
"class_82194","82194","Transcription of documents and other secretarial services"
"class_82195","82195","Provision of mailbox rental and other postal and mailing services"
"class_82196","82196","Photocopying, duplicating, blueprinting"
"class_82199","82199","Photocopying, document preparation and other specialized office support activities n.e.c."
"class_82200","82200","Activities of call centres"
"class_82301","82301","Organization, promotions and/or management of event"
"class_82302","82302","Meeting, incentive, convention, exhibition (MICE)"
"class_82910","82910","Activities of collection agencies and credit bureaus"
"class_82920","82920","Packaging activities on a fee or contract basis, whether or not these involve an automated process"
"class_82990","82990","Other business support service activities n.e.c."
"class_84111","84111","General (overall) public administration activities"
"class_84112","84112","Ancillary service activities for the government as a whole"
"class_84121","84121","Administrative educational services"
"class_84122","84122","Administrative health care services"
"class_84123","84123","Administrative housing and local government services"
"class_84124","84124","Administrative recreational, cultural, arts and sports services"
"class_84125","84125","Administrative religious affairs services"
"class_84126","84126","Administrative welfare services"
"class_84129","84129","Other community and social affairs services"
"class_84131","84131","Domestic and international trade affairs"
"class_84132","84132","Agriculture and rural development affairs"
"class_84133","84133","Primary industries affairs"
"class_84134","84134","Public works affairs"
"class_84135","84135","Transport affairs"
"class_84136","84136","Energy, telecommunication and postal affairs"
"class_84137","84137","Tourism affairs"
"class_84138","84138","Human resource affairs"
"class_84139","84139","Other regulation of and contribution to more efficient operation of businesses n.e.c."
"class_84210","84210","Foreign affairs"
"class_84220","84220","Military and civil defence services"
"class_84231","84231","Police service"
"class_84232","84232","Prison service"
"class_84233","84233","Immigration service"
"class_84234","84234","National registration service"
"class_84235","84235","Judiciary and legal service"
"class_84236","84236","Firefighting and fire prevention"
"class_84239","84239","Other public order and safety affairs related services"
"class_84300","84300","Compulsory social security activities e.g. SOCSO"
"class_85101","85101","Pre-primary education (Public)"
"class_85102","85102","Pre-primary education (Private)"
"class_85103","85103","Primary education (Public)"
"class_85104","85104","Primary education (Private)"
"class_85211","85211","General school secondary education (Public)"
"class_85212","85212","General school secondary education (Private)"
"class_85221","85221","Technical and vocational education below the level of higher education (Public)"
"class_85222","85222","Technical and vocational education below the level of higher education (Private)"
"class_85301","85301","College and university education (Public)"
"class_85302","85302","College and university education (Private)"
"class_85411","85411","Sports instruction"
"class_85412","85412","Martial arts instruction"
"class_85419","85419","Any other sports and recreation education n.e.c"
"class_85421","85421","Music and dancing school"
"class_85429","85429","Any other cultural education n.e.c."
"class_85491","85491","Tuition centre"
"class_85492","85492","Driving school"
"class_85493","85493","Religious instruction"
"class_85494","85494","Computer training"
"class_85499","85499","Others education n.e.c"
"class_85500","85500","Educational support services for provision of non-instructional services"
"class_86101","86101","Hospital activities"
"class_86102","86102","Maternity home services (outside hospital)"
"class_86201","86201","General medical services"
"class_86202","86202","Specialized medical services"
"class_86203","86203","Dental services"
"class_86901","86901","Dialysis Centres"
"class_86902","86902","Medical laboratories"
"class_86903","86903","Physiotherapy and occupational therapy service"
"class_86904","86904","Acupuncture services"
"class_86905","86905","Herbalist and homeopathy services"
"class_86906","86906","Ambulance services"
"class_86909","86909","Other human health services n.e.c."
"class_87101","87101","Homes for the elderly with nursing care"
"class_87102","87102","Nursing homes"
"class_87103","87103","Palliative or hospices"
"class_87201","87201","Drug rehabilitation centres"
"class_87209","87209","Other residential care activities for mental retardation n.e.c."
"class_87300","87300","Residential care activities for the elderly and disabled"
"class_87901","87901","Orphanages"
"class_87902","87902","Welfare homes services"
"class_87909","87909","Other residential care activities n.e.c."
"class_88101","88101","Day-care activities for the elderly or for handicapped adults"
"class_88109","88109","Others social work activities without accommodation for the elderly and disabled"
"class_88901","88901","Counselling service"
"class_88902","88902","Child day-care activities"
"class_88909","88909","Other social work activities without accommodation n.e.c."
"class_90001","90001","Theatrical producer, singer group band and orchestra entertainment services"
"class_90002","90002","Operation of concert and theatre halls and other arts facilities"
"class_90003","90003","Activities of sculptors, painters, cartoonists, engravers, etchers"
"class_90004","90004","Activities of individual writers, for all subjects"
"class_90005","90005","Activities of independent journalists"
"class_90006","90006","Restoring of works of art such as painting"
"class_90007","90007","Activities of producers or entrepreneurs of arts live events, with or without facilities"
"class_90009","90009","Creative, arts and entertainment activities n.e.c."
"class_91011","91011","Documentation and information activities of libraries of all kinds"
"class_91012","91012","Stock photo libraries and services"
"class_91021","91021","Operation of museums of all kinds"
"class_91022","91022","Operation of historical sites and buildings"
"class_91031","91031","Operation of botanical and zoological gardens"
"class_91032","91032","Operation of nature reserves, including wildlife preservation"
"class_92000","92000","GAMBLING AND BETTING ACTIVITIES"
"class_93111","93111","Football, hockey, cricket, baseball, badminton, futsal, paintball"
"class_93112","93112","Racetracks for auto"
"class_93113","93113","Equestrian clubs"
"class_93114","93114","Swimming pools and stadiums, ice-skating arenas"
"class_93115","93115","Track and field stadium"
"class_93116","93116","Golf courses"
"class_93117","93117","Bowling centre"
"class_93118","93118","Fitness centres"
"class_93119","93119","Organization and operation of outdoor or indoor sports events for professionals or amateurs by organizations with own facilities"
"class_93120","93120","The operation of sports clubs such as football club, bowling club, swimming club"
"class_93191","93191","Activities of producers or promoters of sports events, with or without facilities"
"class_93192","93192","Activities of sports leagues and regulating bodies"
"class_93193","93193","Activities of related to promotion of sporting events"
"class_93199","93199","Other sports activities n.e.c."
"class_93210","93210","Activities of amusement parks and theme parks"
"class_93291","93291","Activities of recreation parks and beaches"
"class_93292","93292","Operation of recreational transport facilities"
"class_93293","93293","Renting of leisure and pleasure equipment as an integral part of recreational facilities"
"class_93294","93294","Operation of fairs and shows of a recreational nature"
"class_93295","93295","Operation of discotheques and dance floors"
"class_93296","93296","Activities of producers or entrepreneurs of live events other than arts or sports events, with or without facilities"
"class_93297","93297","Cyber Café/Internet Centre"
"class_93299","93299","Any other amusement and recreation activities n.e.c."
"class_94110","94110","Activities of business and employers membership organizations"
"class_94120","94120","Activities of professional membership organizations"
"class_94200","94200","Activities of trade unions"
"class_94910","94910","Activities of religious organizations"
"class_94920","94920","Activities of political organizations"
"class_94990","94990","Activities of other membership organizations n.e.c."
"class_95111","95111","Repair of electronic equipment"
"class_95112","95112","Repair and maintenance of computer terminals"
"class_95113","95113","Repair and maintenance of hand-held computers (PDA's)"
"class_95121","95121","Repair and maintenance of cordless telephones"
"class_95122","95122","Repair and maintenance of cellular phones"
"class_95123","95123","Repair and maintenance of carrier equipment modems"
"class_95124","95124","Repair and maintenance of fax machines"
"class_95125","95125","Repair and maintenance of communications transmission equipment"
"class_95126","95126","Repair and maintenance of two-way radios"
"class_95127","95127","Repair and maintenance of commercial TV and video cameras"
"class_95211","95211","Repair and maintenance of television, radio receivers"
"class_95212","95212","Repair and maintenance of VCR/DVD/VCD"
"class_95213","95213","Repair and maintenance of CD players"
"class_95214","95214","Repair and maintenance of household-type video cameras"
"class_95221","95221","Repair and servicing of household appliances"
"class_95222","95222","Repair and servicing of home and garden equipment"
"class_95230","95230","Repair of footwear and leather goods"
"class_95240","95240","Repair of furniture and home furnishings"
"class_95291","95291","Repair of bicycles"
"class_95292","95292","Repair and alteration of clothing"
"class_95293","95293","Repair and alteration of jewellery"
"class_95294","95294","Repair of watches, clocks and their parts"
"class_95295","95295","Repair of sporting goods"
"class_95296","95296","Repair of musical instruments"
"class_95299","95299","Repair of other personal and household goods n.e.c."
"class_96011","96011","Laundering and dry-cleaning, pressing"
"class_96012","96012","Carpet and rug shampooing, and drapery and curtain cleaning, whether on clients' premises or not"
"class_96013","96013","Provision of linens, work uniforms and related items by laundries"
"class_96014","96014","Diaper supply services"
"class_96020","96020","Hairdressing and other beauty treatment"
"class_96031","96031","Preparing the dead for burial or cremation and embalming and morticians' services"
"class_96032","96032","Providing burial or cremation services"
"class_96033","96033","Rental of equipped space in funeral parlours"
"class_96034","96034","Rental or sale of graves"
"class_96035","96035","Maintenance of graves and mausoleums"
"class_96091","96091","Activities of sauna, steam baths, massage salons"
"class_96092","96092","Astrological and spiritualists' activities"
"class_96093","96093","Social activities such as escort services, dating services, services of marriage bureaux"
"class_96094","96094","Pet care services"
"class_96095","96095","Genealogical organizations"
"class_96096","96096","Shoe shiners, porters, valet car parkers"
"class_96097","96097","Concession operation of coin-operated personal service machines"
"class_96099","96099","Other service activities n.e.c."
"class_97000","97000","Activities of households as employers of domestic personnel"
"class_98100","98100","Undifferentiated goods-producing activities of private households for own use"
"class_98200","98200","Undifferentiated service-producing activities of private households for own use"
"class_99000","99000","Activities of extraterritorial organization and bodies"

```

## File: data\my_ubl_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="ubl_21_InvoiceType_my" inherit_id="account_edi_ubl_cii.ubl_21_InvoiceType" primary="True">
        <xpath expr="//*[local-name()='DocumentCurrencyCode']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
               xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cac:AdditionalDocumentReference t-if="vals.get('invoice_incoterm_code')">
                    <cbc:ID t-out="vals['invoice_incoterm_code']"/>
                </cac:AdditionalDocumentReference>
                <cac:AdditionalDocumentReference t-if="vals.get('custom_form_reference')">
                    <cbc:ID t-out="vals['custom_form_reference']"/>
                    <cbc:DocumentType>CustomsImportForm</cbc:DocumentType>
                </cac:AdditionalDocumentReference>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxTotal']" position="before">
            <!-- When applicable, the tax exchange rate MUST be provided. -->
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
               xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cac:TaxExchangeRate t-if="invoice.currency_id.name != 'MYR'">
                    <cbc:SourceCurrencyCode t-out="invoice.currency_id.name"/>
                    <cbc:TargetCurrencyCode>MYR</cbc:TargetCurrencyCode>
                    <cbc:CalculationRate t-out="vals.get('tax_exchange_rate')"/>
                </cac:TaxExchangeRate>
            </t>
        </xpath>
        <!-- MyInvois does not support order references, having one will cause issues -->
        <xpath expr="//*[local-name()='OrderReference']" position="replace"/>
    </template>

    <!-- They are not using the same template at all, so we make a new one. They basically want the same data as supplier/customer party -->
    <template id="ubl_20_DeliveryType_my">
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cac:DeliveryParty>
                <t t-set="accounting_delivery_vals" t-value="vals.get('accounting_delivery_party_vals', {})"/>
                <t t-call="{{PartyType_template}}">
                    <t t-set="vals" t-value="accounting_delivery_vals"/>
                </t>
            </cac:DeliveryParty>
        </t>
    </template>

    <template id="ubl_20_InvoiceLineType_my" inherit_id="account_edi_ubl_cii.ubl_20_InvoiceLineType" primary="True">
        <xpath expr="//*[local-name()='Price']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
               xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cac:ItemPriceExtension>
                    <cbc:Amount t-att-currencyID="vals['currency'].name"
                                t-out="format_float(vals.get('item_price_extension_amount'), vals.get('currency_dp'))"/>
                </cac:ItemPriceExtension>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: data\neutralize.sql

```sql
-- disable l10n_my_edi integration by archiving all proxy users; and reset the mode to pre-production.
UPDATE account_edi_proxy_client_user
   SET active = FALSE
 WHERE proxy_type = 'l10n_my_edi';
UPDATE res_company
   SET l10n_my_edi_mode = 'test';

```

## File: models\account_edi_proxy_user.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from werkzeug.urls import url_join

from odoo import _, fields, models
from odoo.addons.account_edi_proxy_client.models.account_edi_proxy_user import AccountEdiProxyError
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class AccountEdiProxyClientUser(models.Model):
    _inherit = 'account_edi_proxy_client.user'

    # ------------------
    # Fields declaration
    # ------------------

    proxy_type = fields.Selection(selection_add=[('l10n_my_edi', 'Malaysian EDI')], ondelete={'l10n_my_edi': 'cascade'})

    # -----------------------
    # CRUD, inherited methods
    # -----------------------

    def _get_proxy_urls(self):
        # EXTENDS 'account_edi_proxy_client'
        urls = super()._get_proxy_urls()
        # We do not use demo with MyInvois as during a demo, showing the invoice on the pre-prod platform will be better.
        urls['l10n_my_edi'] = {
            'demo': False,
            'prod': 'https://l10n-my-edi.api.odoo.com',
            'test': self.env['ir.config_parameter'].sudo().get_param('l10n_my_edi_test_server_url', 'https://l10n-my-edi.test.odoo.com'),
        }
        return urls

    def _get_proxy_identification(self, company, proxy_type):
        # EXTENDS 'account_edi_proxy_client'
        if proxy_type == 'l10n_my_edi':
            if not company.vat:
                raise UserError(_('Please fill the TIN of company "%(company_name)s" before enabling the integration with MyInvois.',
                                  company_name=company.display_name))
            return company.vat
        return super()._get_proxy_identification(company, proxy_type)

    # ----------------
    # Business methods
    # ----------------

    def _l10n_my_edi_contact_proxy(self, endpoint, params):
        self.ensure_one()
        try:
            response = self._make_request(
                url=url_join(self._get_server_url(), endpoint),
                params=params,
            )
        except AccountEdiProxyError as _error:
            # Request error while contacting the IAP server. We assume it is a temporary error.
            raise UserError(_("Failed to contact the E-Invoicing service. Please try again later."))

        return response

```

## File: models\account_edi_xml_ubl_my.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from datetime import datetime

from pytz import UTC

from odoo import _, api, models
from odoo.addons.account_edi_ubl_cii.models.account_edi_xml_ubl_20 import UBL_NAMESPACES

# Far from ideal, but no better solution yet.
COUNTRY_CODE_MAP = {
    "BD": "BGD", "BE": "BEL", "BF": "BFA", "BG": "BGR", "BA": "BIH", "BB": "BRB", "WF": "WLF", "BL": "BLM", "BM": "BMU",
    "BN": "BRN", "BO": "BOL", "BH": "BHR", "BI": "BDI", "BJ": "BEN", "BT": "BTN", "JM": "JAM", "BV": "BVT", "BW": "BWA",
    "WS": "WSM", "BQ": "BES", "BR": "BRA", "BS": "BHS", "JE": "JEY", "BY": "BLR", "BZ": "BLZ", "RU": "RUS", "RW": "RWA",
    "RS": "SRB", "TL": "TLS", "RE": "REU", "TM": "TKM", "TJ": "TJK", "RO": "ROU", "TK": "TKL", "GW": "GNB", "GU": "GUM",
    "GT": "GTM", "GS": "SGS", "GR": "GRC", "GQ": "GNQ", "GP": "GLP", "JP": "JPN", "GY": "GUY", "GG": "GGY", "GF": "GUF",
    "GE": "GEO", "GD": "GRD", "GB": "GBR", "GA": "GAB", "SV": "SLV", "GN": "GIN", "GM": "GMB", "GL": "GRL", "GI": "GIB",
    "GH": "GHA", "OM": "OMN", "TN": "TUN", "JO": "JOR", "HR": "HRV", "HT": "HTI", "HU": "HUN", "HK": "HKG", "HN": "HND",
    "HM": "HMD", "VE": "VEN", "PR": "PRI", "PS": "PSE", "PW": "PLW", "PT": "PRT", "SJ": "SJM", "PY": "PRY", "IQ": "IRQ",
    "PA": "PAN", "PF": "PYF", "PG": "PNG", "PE": "PER", "PK": "PAK", "PH": "PHL", "PN": "PCN", "PL": "POL", "PM": "SPM",
    "ZM": "ZMB", "EH": "ESH", "EE": "EST", "EG": "EGY", "ZA": "ZAF", "EC": "ECU", "IT": "ITA", "VN": "VNM", "SB": "SLB",
    "ET": "ETH", "SO": "SOM", "ZW": "ZWE", "SA": "SAU", "ES": "ESP", "ER": "ERI", "ME": "MNE", "MD": "MDA", "MG": "MDG",
    "MF": "MAF", "MA": "MAR", "MC": "MCO", "UZ": "UZB", "MM": "MMR", "ML": "MLI", "MO": "MAC", "MN": "MNG", "MH": "MHL",
    "MK": "MKD", "MU": "MUS", "MT": "MLT", "MW": "MWI", "MV": "MDV", "MQ": "MTQ", "MP": "MNP", "MS": "MSR", "MR": "MRT",
    "IM": "IMN", "UG": "UGA", "TZ": "TZA", "MY": "MYS", "MX": "MEX", "IL": "ISR", "FR": "FRA", "IO": "IOT", "SH": "SHN",
    "FI": "FIN", "FJ": "FJI", "FK": "FLK", "FM": "FSM", "FO": "FRO", "NI": "NIC", "NL": "NLD", "NO": "NOR", "NA": "NAM",
    "VU": "VUT", "NC": "NCL", "NE": "NER", "NF": "NFK", "NG": "NGA", "NZ": "NZL", "NP": "NPL", "NR": "NRU", "NU": "NIU",
    "CK": "COK", "XK": "XKX", "CI": "CIV", "CH": "CHE", "CO": "COL", "CN": "CHN", "CM": "CMR", "CL": "CHL", "CC": "CCK",
    "CA": "CAN", "CG": "COG", "CF": "CAF", "CD": "COD", "CZ": "CZE", "CY": "CYP", "CX": "CXR", "CR": "CRI", "CW": "CUW",
    "CV": "CPV", "CU": "CUB", "SZ": "SWZ", "SY": "SYR", "SX": "SXM", "KG": "KGZ", "KE": "KEN", "SS": "SSD", "SR": "SUR",
    "KI": "KIR", "KH": "KHM", "KN": "KNA", "KM": "COM", "ST": "STP", "SK": "SVK", "KR": "KOR", "SI": "SVN", "KP": "PRK",
    "KW": "KWT", "SN": "SEN", "SM": "SMR", "SL": "SLE", "SC": "SYC", "KZ": "KAZ", "KY": "CYM", "SG": "SGP", "SE": "SWE",
    "SD": "SDN", "DO": "DOM", "DM": "DMA", "DJ": "DJI", "DK": "DNK", "VG": "VGB", "DE": "DEU", "YE": "YEM", "DZ": "DZA",
    "US": "USA", "UY": "URY", "YT": "MYT", "UM": "UMI", "LB": "LBN", "LC": "LCA", "LA": "LAO", "TV": "TUV", "TW": "TWN",
    "TT": "TTO", "TR": "TUR", "LK": "LKA", "LI": "LIE", "LV": "LVA", "TO": "TON", "LT": "LTU", "LU": "LUX", "LR": "LBR",
    "LS": "LSO", "TH": "THA", "TF": "ATF", "TG": "TGO", "TD": "TCD", "TC": "TCA", "LY": "LBY", "VA": "VAT", "VC": "VCT",
    "AE": "ARE", "AD": "AND", "AG": "ATG", "AF": "AFG", "AI": "AIA", "VI": "VIR", "IS": "ISL", "IR": "IRN", "AM": "ARM",
    "AL": "ALB", "AO": "AGO", "AQ": "ATA", "AS": "ASM", "AR": "ARG", "AU": "AUS", "AT": "AUT", "AW": "ABW", "IN": "IND",
    "AX": "ALA", "AZ": "AZE", "IE": "IRL", "ID": "IDN", "UA": "UKR", "QA": "QAT", "MZ": "MOZ"
}
# todo This can be removed in master as the codes were updated in the data.
#  But for existing databases, we'll need this as the base module most likely won't get updated.
MALAYSIAN_SUBDIVISION_CODES = {
    "JHR": "MY-01",
    "KDH": "MY-02",
    "KTN": "MY-03",
    "MLK": "MY-04",
    "NSN": "MY-05",
    "PHG": "MY-06",
    "PNG": "MY-07",
    "PRK": "MY-08",
    "PLS": "MY-09",
    "SGR": "MY-10",
    "TRG": "MY-11",
    "SBH": "MY-12",
    "SWK": "MY-13",
    "KUL": "MY-14",
    "LBN": "MY-15",
    "PJY": "MY-16",
}
E_164_REGEX = re.compile(r"^\+[1-9]\d{1,14}$")


class AccountEdiXmlUBLMyInvoisMY(models.AbstractModel):
    """
    * MyInvois API formats doc: https://sdk.myinvois.hasil.gov.my/documents
    """
    _inherit = "account.edi.xml.ubl_21"
    _name = 'account.edi.xml.ubl_myinvois_my'
    _description = "Malaysian implementation of ubl for the MyInvois portal"

    # -----------------------
    # CRUD, inherited methods
    # -----------------------

    def _export_invoice_filename(self, invoice):
        # OVERRIDE 'account_edi_ubl_cii'
        return f"{invoice.name.replace('/', '_')}_myinvois.xml"

    def _export_invoice_vals(self, invoice):
        # EXTENDS 'account_edi_ubl_cii'
        vals = super()._export_invoice_vals(invoice)

        vals.update({
            # MyInvois integration requires some template changes. All documents use the same template (Invoice)
            'InvoiceType_template': 'l10n_my_edi.ubl_21_InvoiceType_my',
            'CreditNoteType_template': 'l10n_my_edi.ubl_21_InvoiceType_my',
            'DebitNoteType_template': 'l10n_my_edi.ubl_21_InvoiceType_my',
            'main_template': 'account_edi_ubl_cii.ubl_20_Invoice',

            'InvoiceLineType_template': 'l10n_my_edi.ubl_20_InvoiceLineType_my',
            'CreditNoteLineType_template': 'l10n_my_edi.ubl_20_InvoiceLineType_my',
            'DebitNoteLineType_template': 'l10n_my_edi.ubl_20_InvoiceLineType_my',

            'DeliveryType_template': 'l10n_my_edi.ubl_20_DeliveryType_my',
        })

        document_type_code, original_document = self._l10n_my_edi_get_document_type_code(invoice)
        vals['vals'].update({
            # These data are not in the API description and thus removed to avoid issues.
            'customization_id': None,
            'profile_id': None,
            'ubl_version_id': None,
            'due_date': None,
            'order_reference': None,
            # The current version is 1.1 (document with signature), the type code depends on the move type.
            'document_type_code_attrs': {'listVersionID': 1.1},
            'document_type_code': document_type_code,
            # The issue time must be the current time set in the UTC time zone
            'issue_time': datetime.now(tz=UTC).strftime("%H:%M:%SZ"),
            # Exchange rate information must be provided if applicable
            'tax_exchange_rate': self._l10n_my_edi_get_tax_exchange_rate(invoice),
            'invoice_incoterm_code': invoice.invoice_incoterm_id.code,
            'custom_form_reference': invoice.l10n_my_edi_custom_form_reference,
        })

        # these are optional, and since we can't have the correct one at the time of generating, we avoid adding them.
        vals['vals'].pop('payment_means_vals_list', None)

        # We add the company industrial classification to the supplier vals.
        vals['vals']['accounting_supplier_party_vals']['party_vals'].update({
            'industry_classification_code_attrs': {'name': invoice.company_id.l10n_my_edi_industrial_classification.name},
            'industry_classification_code': invoice.company_id.l10n_my_edi_industrial_classification.code,
        })
        # We ensure that the customer does not have their ttx set (it could be on the record if they're also supplier)
        customer_identification_vals = [
            vals for vals in vals['vals']['accounting_customer_party_vals']['party_vals']['party_identification_vals'] if vals['id_attrs'] != {'schemeID': 'TTX'}
        ]
        vals['vals']['accounting_customer_party_vals']['party_vals']['party_identification_vals'] = customer_identification_vals

        # Debit/Credit note original invoice ref.
        if original_document:
            vals['vals'].update({
                'billing_reference_vals': {
                    'id': original_document.name,
                    'uuid': original_document.l10n_my_edi_external_uuid,
                },
            })

        return vals

    def _get_delivery_vals_list(self, invoice):
        # OVERRIDE 'account_edi_ubl_cii'
        return [{
            'accounting_delivery_party_vals': self._l10n_my_edi_get_delivery_party_vals(invoice.partner_id),
        }]

    def _get_partner_contact_vals(self, partner):
        # EXTENDS 'account_edi_ubl_cii'
        res = super()._get_partner_contact_vals(partner)
        res['telephone'] = self._l10n_my_edi_get_formatted_phone_number(res['telephone'])
        return res

    def _get_country_vals(self, country):
        # EXTENDS 'account_edi_ubl_cii'
        vals = super()._get_country_vals(country)
        vals.update({
            'identification_code_attrs': {
                'listID': 'ISO3166-1',
                'listAgencyID': '6',
            },
            'identification_code': COUNTRY_CODE_MAP.get(country.code),
        })
        return vals

    def _get_partner_address_vals(self, partner):
        # EXTENDS 'account_edi_ubl_cii'
        vals = super()._get_partner_address_vals(partner)
        # We do not want to display the streets, but instead use the AddressLine element.
        vals.pop('street_name', None)
        vals.pop('additional_street_name', None)

        # The API expects the iso3166-2 code for the state, in the same way as it expects the iso3166 code for the countries.
        # In Odoo, we mostly use these (although there is no standard format) so we'll try to use what Odoo gives us.
        # For malaysia, the codes where updated, but we use a mapping to ensure that outdated data will still end up correct.
        subentity_code = partner.state_id.code or ''

        if partner.country_id.code == 'MY' and partner.state_id.code in MALAYSIAN_SUBDIVISION_CODES:
            subentity_code = MALAYSIAN_SUBDIVISION_CODES[partner.state_id.code]

        # The API does not expect the country code inside the state code, only the number part.
        if f'{partner.country_id.code}-' in subentity_code:
            subentity_code = subentity_code.split('-')[1]

        vals.update({
            'address_lines': [partner.street or '', partner.street2 or ''],
            'country_subentity_code': subentity_code,
        })
        return vals

    def _get_partner_party_legal_entity_vals_list(self, partner):
        # OVERRIDE 'account_edi_ubl_cii'
        # We only want to display the registration name here.
        return [{
            'registration_name': partner.name,
        }]

    def _get_partner_party_identification_vals_list(self, partner):
        """ The id vals list must be filled with two values.
        The TIN, and then one of either:
            - Business registration number (BNR)
            - MyKad/MyTentera identification number (NRIC)
            - Passport number or MyPR/MyKAS identification number (PASSPORT)
            - (ARMY)
        Additionally, companies registered to use SST (sales & services tax) must provide their SST number.
        Finally, if a supplier is using TTX (tourism tax), once again that number must be provided.
        """
        # EXTENDS 'account_edi_ubl_cii'
        vals = super()._get_partner_party_identification_vals_list(partner)

        vals.append({
            'id_attrs': {'schemeID': 'TIN'},
            'id': partner._l10n_my_edi_get_tin_for_myinvois(),
        })

        if partner.l10n_my_identification_type and partner.l10n_my_identification_number:
            vals.append({
                'id_attrs': {'schemeID': partner.l10n_my_identification_type},
                'id': partner.l10n_my_identification_number,
            })
            if partner.sst_registration_number:
                # The supplier can input up to 2 SST numbers, in which case they need to separate both by a ;
                # They can do so in the existing field if they want.
                vals.append({
                    'id_attrs': {'schemeID': 'SST'},
                    'id': partner.sst_registration_number,
                })
            if partner.ttx_registration_number:
                vals.append({
                    'id_attrs': {'schemeID': 'TTX'},
                    'id': partner.ttx_registration_number,
                })
        return vals

    def _get_partner_party_tax_scheme_vals_list(self, partner, role):
        """ This information is not needed. Instead, the party identification vals must be filled. """
        # OVERRIDE 'account_edi_ubl_cii'
        return []

    def _get_tax_unece_codes(self, customer, supplier, tax):
        # OVERRIDE 'account_edi_ubl_cii'
        return {
            'tax_category_code': tax.l10n_my_tax_type,
            'tax_exemption_reason_code': None,  # Unused in this file.
            'tax_exemption_reason': None,  # Should be set here but we no longer have access to the invoice info...
        }

    def _get_tax_category_list(self, customer, supplier, taxes):
        # EXTENDS 'account_edi_ubl_cii'
        vals_list = super()._get_tax_category_list(customer, supplier, taxes)

        for vals in vals_list:
            vals['tax_scheme_vals']['id'] = 'OTH'
            vals['tax_scheme_vals']['id_attrs'] = {'schemeID': 'UN/ECE 5153', 'schemeAgencyID': '6'}

        return vals_list

    def _export_invoice_constraints(self, invoice, vals):
        # EXTENDS 'account_edi_ubl_cii'
        constraints = super()._export_invoice_constraints(invoice, vals)

        # In malaysia, tax on good is paid at the manufacturer level. It is thus common to invoice without taxes,
        # unless invoicing for a service.
        constraints.pop('tax_on_line', '')
        constraints.pop('cen_en16931_tax_line', '')

        if not invoice.company_id.l10n_my_edi_industrial_classification:
            self._l10n_my_edi_make_validation_error(constraints, 'industrial_classification_required', 'company', invoice.company_id.display_name)

        for partner_type in ('supplier', 'customer'):
            partner = vals[partner_type]
            phone_number = partner.phone or partner.mobile
            if phone_number:
                phone = self._l10n_my_edi_get_formatted_phone_number(phone_number)
                if E_164_REGEX.match(phone) is None:
                    self._l10n_my_edi_make_validation_error(constraints, 'phone_number_format', partner_type, partner.display_name)
            else:
                self._l10n_my_edi_make_validation_error(constraints, 'phone_number_required', partner_type, partner.display_name)

            # We need to provide both l10n_my_identification_type and l10n_my_identification_number
            if not partner.commercial_partner_id.l10n_my_identification_type or not partner.commercial_partner_id.l10n_my_identification_number:
                self._l10n_my_edi_make_validation_error(constraints, 'required_id', partner_type, partner.commercial_partner_id.display_name)

            if not partner.state_id:
                self._l10n_my_edi_make_validation_error(constraints, 'no_state', partner_type, partner.display_name)
            if not partner.city:
                self._l10n_my_edi_make_validation_error(constraints, 'no_city', partner_type, partner.display_name)
            if not partner.country_id:
                self._l10n_my_edi_make_validation_error(constraints, 'no_country', partner_type, partner.display_name)
            if not partner.street:
                self._l10n_my_edi_make_validation_error(constraints, 'no_street', partner_type, partner.display_name)

            if partner.commercial_partner_id.sst_registration_number and len(partner.commercial_partner_id.sst_registration_number.split(';')) > 2:
                self._l10n_my_edi_make_validation_error(constraints, 'too_many_sst', partner_type, partner.commercial_partner_id.display_name)

        for line in invoice.invoice_line_ids.filtered(lambda line: line.display_type not in ('line_note', 'line_section')):
            if line.product_id and not line.product_id.product_tmpl_id.l10n_my_edi_classification_code:
                self._l10n_my_edi_make_validation_error(constraints, 'class_code_required', line.product_id.id, line.product_id.display_name)
            if not line.tax_ids:
                self._l10n_my_edi_make_validation_error(constraints, 'tax_ids_required', line.id, line.display_name)
            elif any(tax.l10n_my_tax_type == 'E' for tax in line.tax_ids) and not invoice.l10n_my_edi_exemption_reason:
                self._l10n_my_edi_make_validation_error(constraints, 'tax_exemption_required', invoice.id, invoice.display_name)

        document_type_code, original_document = self._l10n_my_edi_get_document_type_code(invoice)
        if document_type_code != '01' and not original_document:
            self._l10n_my_edi_make_validation_error(constraints, 'adjustment_origin', invoice.id, invoice.display_name)

        return constraints

    def _get_invoice_line_item_vals(self, line, taxes_vals):
        # EXTENDS 'account_edi_ubl_cii'
        vals = super()._get_invoice_line_item_vals(line, taxes_vals)
        vals['commodity_classification_vals'] = [{
            'item_classification_code': line.product_id.product_tmpl_id.l10n_my_edi_classification_code,
            'item_classification_attrs': {'listID': 'CLASS'},
        }]
        # User the tax_details in order to fill the classified_tax_category_vals as would be expected.
        for tax_detail in taxes_vals['tax_details']:
            tax_category_vals = tax_detail['_tax_category_vals_']
            for classified_tax_category_vals in vals['classified_tax_category_vals']:
                if tax_category_vals['id'] == classified_tax_category_vals['id']:
                    classified_tax_category_vals['name'] = tax_category_vals['name']
                    classified_tax_category_vals['tax_exemption_reason'] = tax_category_vals['tax_exemption_reason']

        return vals

    def _get_tax_grouping_key(self, base_line, tax_data):
        # EXTENDS 'account_edi_ubl_cii'
        grouping_key = super()._get_tax_grouping_key(base_line, tax_data)
        # Add the tax exemption here as well to ensure consistency.
        tax = tax_data['tax']
        invoice = base_line['record'].move_id
        grouping_key['_tax_category_vals_']['name'] = invoice.l10n_my_edi_exemption_reason if tax.l10n_my_tax_type == 'E' else None
        grouping_key['_tax_category_vals_']['tax_exemption_reason'] = invoice.l10n_my_edi_exemption_reason if tax.l10n_my_tax_type == 'E' else None
        return grouping_key

    def _get_invoice_line_vals(self, line, line_id, taxes_vals):
        # EXTENDS 'account_edi_ubl_cii'
        vals = super()._get_invoice_line_vals(line, line_id, taxes_vals)
        vals['item_price_extension_amount'] = line.price_subtotal
        return vals

    def _import_retrieve_partner_vals(self, tree, role):
        """ Returns a dict of values that will be used to retrieve the partner """
        # EXTENDS 'account_edi_ubl_cii'
        vals = super()._import_retrieve_partner_vals(tree, role)
        # We invert the country map to get the country code.
        country_map = {v: k for k, v in COUNTRY_CODE_MAP.items()}
        # We can't use _find_value for the identifier since we need to get the attribute.
        vals.update({
            # Update some values to be correct.
            'vat': self._find_value(f'.//cac:Accounting{role}Party/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID="TIN"]', tree),
            'country_code': country_map.get(self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cac:Country//cbc:IdentificationCode', tree)),
            'name': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:RegistrationName', tree),
            # And add new ones that are expected.
            'sst': self._find_value(f'.//cac:Accounting{role}Party/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID="SST"]', tree),
            'ttx': self._find_value(f'.//cac:Accounting{role}Party/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID="TTX"]', tree),
        })

        identifier = tree.xpath(f'.//cac:Accounting{role}Party/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID="NRIC" or @schemeID="PASSPORT" or @schemeID="BRN" or @schemeID="ARMY"]', namespaces=UBL_NAMESPACES)
        if identifier:  # Technically it's required, but to be safe...
            vals.update({
                'id_type': identifier[0].attrib['schemeID'],
                'id_val': identifier[0].text,
            })

        return vals

    def _import_retrieve_and_fill_partner(self, invoice, name, phone, mail, vat, country_code, id_type, id_val, sst=False, ttx=False):
        """ In addition to the basic values, we need to fill the identifiers of the partner and eventual tax codes. """
        # OVERRIDE 'account_edi_ubl_cii'

        # I consider that the standard _retrieve_partner should be enough to match.
        invoice.partner_id = self.env['res.partner'].with_company(invoice.company_id)._retrieve_partner(name=name, phone=phone, mail=mail, vat=vat)

        if not invoice.partner_id and name and vat:
            partner_vals = {
                'name': name,
                'email': mail,
                'phone': phone,
                'sst_registration_number': sst,
                'ttx_registration_number': ttx,
                'l10n_my_identification_type': id_type,
                'l10n_my_identification_number': id_val,
            }
            country = self.env.ref(f'base.{country_code.lower()}', raise_if_not_found=False)
            if country:
                partner_vals['country_id'] = country.id
            invoice.partner_id = self.env['res.partner'].create(partner_vals)
            if vat and self.env['res.partner']._run_vat_test(vat, country, invoice.partner_id.is_company):
                invoice.partner_id.vat = vat

    def _import_fill_invoice_form(self, invoice, tree, qty_factor):
        # EXTENDS 'account_edi_ubl_cii'
        logs = super()._import_fill_invoice_form(invoice, tree, qty_factor)
        # We get the incoterm
        incoterm_code = self._find_value('./cac:AdditionalDocumentReference[not(descendant::cbc:DocumentType)]/cbc:ID', tree)
        if incoterm_code is not None:
            invoice.invoice_incoterm_id = self.env['account.incoterms'].search([('code', '=', incoterm_code)], limit=1)
        custom_form_ref = self._find_value('./cac:AdditionalDocumentReference[descendant::cbc:DocumentType[text()="CustomsImportForm"]]/cbc:ID', tree)
        invoice.l10n_my_edi_custom_form_reference = custom_form_ref

        # So that we can find the original invoice in case of debit/credit note.
        invoice_type = self._find_value('./cbc:InvoiceTypeCode', tree)
        origin_uuid = self._find_value('.//cac:InvoiceDocumentReference[descendant::cbc:ID[text()="Document Internal ID"]]/cbc:UUID', tree)
        if invoice_type == '02':
            invoice.reversed_entry_id = self.env['account.move'].search([('l10n_my_edi_external_uuid', '=', origin_uuid)], limit=1)
        elif invoice_type == '03' and 'debit_origin_id' in self.env['account.move']._fields:
            invoice.debit_origin_id = self.env['account.move'].search([('l10n_my_edi_external_uuid', '=', origin_uuid)], limit=1)
        return logs

    # ----------------
    # Business methods
    # ----------------

    def _l10n_my_edi_get_delivery_party_vals(self, partner):
        """ Returns the vals required to display the delivery information in the invoice. """
        return {
            'partner': partner,
            'party_identification_vals': self._get_partner_party_identification_vals_list(partner.commercial_partner_id),
            'postal_address_vals': self._get_partner_address_vals(partner),
            'party_legal_entity_vals': self._get_partner_party_legal_entity_vals_list(partner.commercial_partner_id),
        }

    @api.model
    def _l10n_my_edi_get_document_type_code(self, invoice):
        """ Returns the code matching the invoice type, as well as the original document if any. """
        if 'debit_origin_id' in self.env['account.move']._fields and invoice.debit_origin_id:
            return '03', invoice.debit_origin_id
        elif invoice.move_type == 'out_refund':
            return '02', invoice.reversed_entry_id
        else:
            return '01', None

    @api.model
    def _l10n_my_edi_get_tax_exchange_rate(self, invoice):
        """ Returns the tax exchange rate if applicable. We will compute it based on the invoice totals.
        This should be the rate to convert a foreign currency into MYR.
        """
        if invoice.currency_id.name != "MYR":
            # I couldn't find any information on maximum precision, so we will use the currency format.
            return self.env.ref('base.MYR').round(abs(invoice.amount_total_signed) / (invoice.amount_total or 1))
        return ''

    @api.model
    def _l10n_my_edi_get_formatted_phone_number(self, number):
        # the phone number MUST follow the E.164 format.
        # Don't try to reformat too much, we don't want to risk messing it up
        if not number:
            return ''  # This wouldn't happen in the file as it's caught in the validation errors, but the vals are exported before these checks are done.
        return number.replace(' ', '').replace('(', '').replace(')', '').replace('-', '')

    @api.model
    def _l10n_my_edi_make_validation_error(self, constraints, code, record_identifier, record_name):
        """ Small helper that add new constrains into provided constrains dict.
        This helper is mainly there to keep the check method tidy, and focused on its purpose (validating data)
        """
        message_mapping = {
            'industrial_classification_required': _(
                "The industrial classification must be defined on company: %(company_name)s",
                company_name=record_name
            ),
            'phone_number_format': _(
                "The following partner's phone number should follow the E.164 format: %(partner_name)s",
                partner_name=record_name
            ),
            'phone_number_required': _(
                "The following partner's phone number is missing: %(partner_name)s",
                partner_name=record_name)
            ,
            'required_id': _(
                "The following partner's identification type or number is missing: %(partner_name)s",
                partner_name=record_name
            ),
            'no_state': _(
                "The following partner's state is missing: %(partner_name)s",
                partner_name=record_name
            ),
            'no_city': _(
                "The following partner's city is missing: %(partner_name)s",
                partner_name=record_name
            ),
            'no_country': _(
                "The following partner's country is missing: %(partner_name)s",
                partner_name=record_name
            ),
            'no_street': _(
                "The following partner's street is missing: %(partner_name)s",
                partner_name=record_name
            ),
            'class_code_required': _(
                "The following product must have their item classification code set: %(product_name)s",
                product_name=record_name
            ),
            'class_code_required_line': _(
                "The following line must have their item classification code set: %(line_name)s",
                line_name=record_name
            ),
            'adjustment_origin': _(
                "You cannot send a debit / credit note for invoice %(invoice_number)s as it has not yet been sent to MyInvois.",
                invoice_number=record_name
            ),
            'too_many_sst': _(
                "The following partner's should have at most two SST numbers, separated by a semicolon : %(partner_name)s",
                partner_name=record_name
            ),
            'tax_ids_required': _(
                "You must set a tax on the line : %(line_name)s.\nIf taxes are not applicable, please set a 0%% tax with a tax type 'Not Applicable'.",
                line_name=record_name
            ),
            'tax_exemption_required': _(
                "You must set a Tax Exemption Reason on the invoice : %(invoice_name)s as some taxes have the type 'Tax exemption'.",
                invoice_name=record_name
            ),
        }

        constraints[f'myinvois_{record_identifier}_{code}'] = message_mapping[code]

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import datetime
import logging
import time

import dateutil

from odoo import _, api, fields, models
from odoo.exceptions import UserError
from odoo.osv import expression
from odoo.tools import split_every

_logger = logging.getLogger(__name__)

# Holds the maximum amount of invoices that can be sent in a single submission. Should most likely not change.
# Using a constant makes it easy to patch during testing to avoid needing to create 100+ invoices.
SUBMISSION_MAX_SIZE = 100
MAX_SUBMISSION_UPDATE = 25
# An invalid invoice is considered as cancelled by the platform.
CANCELLED_STATES = {'invalid', 'cancelled'}

NAMESPACES = {
    'cac': 'urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2',
    'cbc': 'urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2',
    None: 'urn:oasis:names:specification:ubl:schema:xsd:Invoice-2',
}


class AccountMove(models.Model):
    _inherit = "account.move"

    # ------------------
    # Fields declaration
    # ------------------

    l10n_my_edi_file_id = fields.Many2one(
        comodel_name='ir.attachment',
        compute=lambda self: self._compute_linked_attachment_id('l10n_my_edi_file_id', 'l10n_my_edi_file'),
        depends=['l10n_my_edi_file'],
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    l10n_my_edi_file = fields.Binary(
        string='MyInvois XML File',
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    l10n_my_edi_display_tax_exemption_reason = fields.Boolean(
        compute='_compute_l10n_my_edi_display_tax_exemption_reason',
        string="Display Tax Exemption Reason",
        export_string_translation=False,
    )
    l10n_my_edi_exemption_reason = fields.Char(
        string="Tax Exemption Reason",
        help="Buyer’s sales tax exemption certificate number, special exemption as per gazette orders, etc.\n"
             "Only applicable if you are using a tax with a type 'Exempt'.",
    )
    l10n_my_edi_custom_form_reference = fields.Char(
        string="Customs Form Reference Number",
        help="Reference Number of Customs Form No.1, 9, etc.",
    )
    # False => Not sent yet.
    l10n_my_edi_state = fields.Selection(
        string='MyInvois State',
        help='State of this invoice on the MyInvois portal.\nAn invoice awaiting validation will be automatically updated once the validation status is available.',
        selection=[
            ('in_progress', 'Validation In Progress'),
            ('valid', 'Valid'),
            ('rejected', 'Rejected'),  # Technically not a state on MyInvois, but having it here helps with managing bills.
            ('invalid', 'Invalid'),
            ('cancelled', 'Cancelled'),
        ],
        copy=False,
        readonly=True,
        tracking=True,
        export_string_translation=False,
    )
    # Users have 72h after the validation of an invoice to cancel it. Passed that time, they need to issue a credit or debit note.
    l10n_my_edi_validation_time = fields.Datetime(
        string='Validation Time',
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    l10n_my_edi_submission_uid = fields.Char(
        string='Submission UID',
        help="Unique ID assigned to a batch of invoices when sent to MyInvois.",
        copy=False,
        readonly=True,
    )
    l10n_my_edi_external_uuid = fields.Char(
        string="MyInvois ID",
        help="Unique ID assigned to a specific invoice when sent to MyInvois.",
        copy=False,
        index=True,
        readonly=True,
    )
    # In case of error, we will use the hash as they ask to avoid resending identical invoice.
    l10n_my_error_document_hash = fields.Char(
        string="Document Hash",
        copy=False,
        readonly=True,
        export_string_translation=False,
    )
    l10n_my_edi_retry_at = fields.Char(
        string="Document Retry At",
        copy=False,
        readonly=True,
        export_string_translation=False,
    )

    # --------------------------------
    # Compute, inverse, search methods
    # --------------------------------

    @api.depends('l10n_my_edi_state')
    def _compute_need_cancel_request(self):
        # EXTENDS 'account'
        super()._compute_need_cancel_request()

    @api.depends('l10n_my_edi_state')
    def _compute_show_reset_to_draft_button(self):
        # EXTEND 'account'
        super()._compute_show_reset_to_draft_button()
        self.filtered(lambda m: m.l10n_my_edi_state and m.l10n_my_edi_state not in CANCELLED_STATES).show_reset_to_draft_button = False

    @api.depends('company_id', 'invoice_line_ids.tax_ids')
    def _compute_l10n_my_edi_display_tax_exemption_reason(self):
        """ Some users will never use tax-exempt taxes, so it's better to only show the field when necessary. """
        for move in self:
            should_display = move._l10n_my_edi_uses_edi() and any(tax.l10n_my_tax_type == 'E' for tax in move.invoice_line_ids.tax_ids)
            move.l10n_my_edi_display_tax_exemption_reason = should_display

    # -----------------------
    # CRUD, inherited methods
    # -----------------------

    def button_request_cancel(self):
        # EXTENDS 'account'
        super().button_request_cancel()

        if self._need_cancel_request() and self.l10n_my_edi_state in ['valid', 'rejected']:
            self._l10n_my_edi_check_can_update_status()

            return {
                "name": _("Cancel Document"),
                "type": "ir.actions.act_window",
                "view_type": "form",
                "view_mode": "form",
                "res_model": "l10n_my_edi.document.status.update",
                "target": "new",
                "context": {
                    "default_invoice_id": self.id,
                    "default_new_status": 'cancelled',
                },
            }

        return super().button_request_cancel()

    def button_draft(self):
        # EXTENDS 'account'

        # If the invoice has been completely cancelled, we allow resetting to draft to ease the process to reissue the invoice.
        # Not that it may be preferable to leave the invoice as cancelled, and issue a new one instead.
        invoices_to_reset = self.filtered(
            lambda i: (i.state == 'cancel' and i.l10n_my_edi_state in CANCELLED_STATES)
        )
        res = super().button_draft()
        # We do not reset the hash and retry time, as an invalid invoice that is being re-sent must be modified (hash should change)
        invoices_to_reset.write({
            'l10n_my_edi_state': False,
            'l10n_my_edi_validation_time': False,
            'l10n_my_edi_submission_uid': False,
            'l10n_my_edi_external_uuid': False,
        })
        invoices_to_reset.l10n_my_edi_file_id.unlink()
        return res

    def _get_fields_to_detach(self):
        # EXTENDS account
        fields_list = super()._get_fields_to_detach()
        fields_list.append('l10n_my_edi_file')
        return fields_list

    def _need_cancel_request(self):
        # EXTENDS 'account'
        # For the in_progress state, we do not want to allow resetting to draft nor cancelling. We need to wait for the result first.
        return super()._need_cancel_request() or self.l10n_my_edi_state in ['valid', 'rejected']

    # --------------
    # Action methods
    # --------------

    def action_l10n_my_edi_update_status(self):
        self.ensure_one()
        result = self._l10n_my_edi_fetch_status()

        # This is called manually by the user. In case of errors, we will raise.
        # If the validation failed or the invoice has been rejected/cancelled, we will log the result in the chatter.
        if 'error' in result:
            raise UserError(self._l10n_my_edi_map_error(result['error']))

        # If there has been no status change, we do not want to do anything.
        if result['status'] == self.l10n_my_edi_state:
            return

        if 'validation_errors' in result:
            validation_error = self.env['account.move.send']._format_error_html({
                'error_title': _('The validation failed with the following errors:'),
                'errors': result['validation_errors'],
            })
            self._l10n_my_edi_set_status(result['status'], validation_error)
        elif result.get('status_reason'):
            self._l10n_my_edi_set_status(
                result['status'],
                message=_('This invoice has been %(status)s for reason: %(reason)s', status=result['status'], reason=result['status_reason']),
            )
        else:
            self._l10n_my_edi_set_status(result['status'])

        # As done during submission flow, when the status becomes
        if self.l10n_my_edi_state == 'valid':
            self._update_validation_fields(result)

    def action_l10n_my_edi_reject_bill(self):
        self.ensure_one()

        if self.l10n_my_edi_state == "valid":
            self._l10n_my_edi_check_can_update_status()

            return {
                "name": _("Reject Document"),
                "type": "ir.actions.act_window",
                "view_type": "form",
                "view_mode": "form",
                "res_model": "l10n_my_edi.document.status.update",
                "target": "new",
                "context": {
                    "default_invoice_id": self.id,
                    "default_new_status": 'rejected',
                },
            }

    def action_validate_tin(self):
        self.ensure_one()
        self.partner_id.action_validate_tin()

    # ----------------
    # Business methods
    # ----------------

    # API methods

    def _l10n_my_edi_submit_documents(self, xml_contents):
        """ Contact our IAP service in order to send the invoice xml to the MyInvois API. """
        proxy_user = self._l10n_my_edi_ensure_proxy_user()

        # We really only care about moves that appears in the xml contents.
        moves_to_send = self.filtered(lambda move: move in xml_contents)
        if not moves_to_send:
            return None

        # Ensure to lock the records that will be sent, to avoid risking sending them twice.
        self.env['res.company']._with_locked_records(moves_to_send)

        errors = {}
        success_messages = {}
        move_to_cancel = self.env['account.move']

        # MyInvois only supports up to 100 invoice per submission. To avoid timing out on big batches, we split it client side.
        for move_batch in split_every(SUBMISSION_MAX_SIZE, moves_to_send.ids, self.env['account.move'].browse):
            move_per_id = {move.id: move for move in move_batch}
            batch_result = proxy_user._l10n_my_edi_contact_proxy(
                endpoint='api/l10n_my_edi/1/submit_invoices',
                params={
                    'documents': [{
                        'move_id': move.id,
                        'move_name': move.name,
                        'error_document_hash': move.l10n_my_error_document_hash,
                        'retry_at': move.l10n_my_edi_retry_at,
                        'data': base64.b64encode(xml_contents[move].encode()).decode(),
                    } for move in move_batch]
                }
            )

            # If an error is present in the result itself (and not per invoice), it means that the whole submission failed.
            # We don't add to the result but instead directly in the errors.
            if 'error' in batch_result:
                error_string = self._l10n_my_edi_map_error(batch_result['error'])
                errors.update({move: [error_string] for move in move_batch})
            else:
                for document_result in batch_result['documents']:
                    move = move_per_id[document_result['move_id']]
                    success = document_result['success']

                    updated_values = {
                        'l10n_my_edi_external_uuid': document_result.get('uuid'),  # rejected documents do not have a uuid.
                        'l10n_my_edi_submission_uid': batch_result['submission_uid'],
                        'l10n_my_edi_state': 'in_progress' if success else 'invalid',
                    }

                    if success:
                        # Ids are logged for future references. An invalid invoice may be reset to resend it after correction, which would be a new submission/uuid.
                        success_messages[move.id] = _('The invoice has been sent to MyInvois with uuid "%(uuid)s" and submission id "%(submission_id)s".\nValidation results will be available shortly.',
                                                      uuid=document_result['uuid'], submission_id=batch_result['submission_uid'])
                    else:
                        # When we raise a "hash_resubmitted" error, we don't resend the same hash/retry at and don't want to rewrite.
                        if 'error_document_hash' in document_result:
                            updated_values.update({
                                'l10n_my_error_document_hash': document_result['error_document_hash'],
                                'l10n_my_edi_retry_at': document_result['retry_at'],
                            })
                        errors[move] = [self._l10n_my_edi_map_error(error) for error in document_result['errors']]
                        move_to_cancel |= move

                    move.write(updated_values)

            if self._can_commit():
                self._cr.commit()

        # For successful moves, we log the sending here. Any errors will be handled by the send & print wizard.
        if success_messages:
            self.env['account.move'].browse(list(success_messages.keys()))._message_log_batch(
                bodies=success_messages,
            )

        if move_to_cancel:
            # Invalid moves should be considered as cancelled; they need to be reset to draft, corrected and sent again.
            move_to_cancel._l10n_my_edi_cancel_moves()

        return errors

    def _l10n_my_edi_fetch_updated_statuses(self):
        """
        Contact our IAP service in order to get the status of the invoices in self.
        Statuses are fetched in batches using the l10n_my_edi_submission_uid field.
        One batch is at most 100 invoices.

        Note that this is only expected to be used during the submission flow, and not later.
        """
        proxy_user = self._l10n_my_edi_ensure_proxy_user()

        self.env['res.company']._with_locked_records(self)

        errors = {}
        any_in_progress = False
        invalid_moves = self.env['account.move']
        for submission_uid, move_batch in self.grouped('l10n_my_edi_submission_uid').items():
            if not submission_uid:
                continue  # While it should never happen, it does not hurt to ensure that we won't try anything in such cases.

            error, statuses = self._l10n_my_get_submission_status(submission_uid, proxy_user)

            if error:
                errors.update({move: [error] for move in move_batch})
                continue

            for move in move_batch:
                status_info = statuses.get(move.l10n_my_edi_external_uuid)
                # If the status did not change, we do not need to do anything.
                if not status_info or move.l10n_my_edi_state == status_info['status']:
                    continue

                move.l10n_my_edi_state = status_info['status']
                if move.l10n_my_edi_state == 'invalid':
                    invalid_moves |= move
                    # Most of the time no reason is provided, but this is not useful. So we will fetch the exact errors individually.
                    if status_info.get('reason'):
                        errors[move] = [_('The MyInvois platform returned an "Invalid" status for this invoice for reason: %(reason)s', reason=status_info['reason'])]
                    else:
                        result = move._l10n_my_edi_fetch_status()
                        if 'error' in result:
                            errors[move] = [self._l10n_my_edi_map_error(result['error'])]
                        elif 'validation_errors' in result:
                            errors[move] = [self.env['account.move.send']._format_error_html({
                                'error_title': _('The validation failed with the following errors:'),
                                'errors': result['validation_errors'],
                            })]
                        elif result['status_reason']:
                            errors[move] = [result['status_reason']]
                elif move.l10n_my_edi_state == 'valid':
                    move._update_validation_fields(status_info)

            if self._can_commit():
                self._cr.commit()

        # We don't consider these errors per-say. From my understanding an invalid invoice is considered as cancelled,
        # so a new one must be issued.
        # For ease of use, we allow an invalid invoice to be reset to draft, but this will erase all links to the previously
        # cancelled invoice.
        if errors:
            # According to their documentation, you cannot cancel an already invalid invoice (they are considered cancelled by default)
            # It makes sense to consider these cancelled in Odoo too, for simplicity.
            invalid_moves._l10n_my_edi_cancel_moves()

        # Invalid or in progress invoices must return errors to stop the email sending/...
        return errors, any_in_progress

    def _l10n_my_edi_update_document(self, status, reason):
        """ Sent invoices can be cancelled, and received bills can be rejected up to 72h after validation.

        This method will try to update the status of a document on the platform, and if needed also the status in Odoo.

        There is no "Rejected" status on the platform. The document stays as 'valid' until action is taken by the vendor.
        At that point, the invoice will be cancelled if need be by the call to _l10n_my_edi_set_status.
        """
        self.ensure_one()
        self.env['res.company']._with_locked_records(self)
        proxy_user = self._l10n_my_edi_ensure_proxy_user()

        # While we do this check before opening the wizard (to avoid filling the wizard for nothing), it is safer to
        # recheck here in case we exceeded the limit in the meantime or if this is called from elsewhere.
        self._l10n_my_edi_check_can_update_status()

        successfully_updated_invoices = self.env['account.move']
        for document in self:
            result = proxy_user._l10n_my_edi_contact_proxy(
                endpoint='api/l10n_my_edi/1/update_status',
                params={
                    'status_values': {
                        'uuid': document.l10n_my_edi_external_uuid,
                        'reason': reason,
                        'status': status,
                    },
                },
            )

            # If it is not a success, it will have raised an error.
            if 'error' in result:
                self._message_log(body=self._l10n_my_edi_map_error(result['error']))
            else:
                successfully_updated_invoices |= document

        if status in self._fields['l10n_my_edi_state'].get_values(self.env):
            successfully_updated_invoices._l10n_my_edi_set_status(
                state=status,
                message=_('This invoice has been %(status)s for reason: %(reason)s', status=status, reason=reason),
            )

        if self._can_commit():
            self._cr.commit()

    @api.model
    def _cron_l10n_my_edi_synchronize_myinvois(self):
        """
        This cron is based on the recommended method to fetch the status of the documents according to their doc.
        MAX_SUBMISSION_UPDATE defines how many submissions to process in a single cron run.
        """
        # First step is to get the invoices for which the status is not yet final.
        # A invoice whose status will not change anymore is: (cancelled or invalid) or has been validated more than 74h ago.
        # /!\ when an invoice validation is pending, l10n_my_edi_validation_time is still None. These also need to be updated.
        datetime_threshold = datetime.datetime.now() - datetime.timedelta(hours=74)
        # We always want to fetch in_progress invoices, it's very likely that their status is already there.
        invoice_domain = [("l10n_my_edi_state", "=", "in_progress")]
        # For valid invoices, we want them if their l10n_my_edi_validation_time is less than 74h ago, and if their l10n_my_edi_retry_at in the past.
        invoice_domain = expression.OR([invoice_domain, [
            ('l10n_my_edi_state', '=', 'valid'),
            ('l10n_my_edi_validation_time', '>', datetime_threshold),
            '|',
            ('l10n_my_edi_retry_at', '<=', datetime.datetime.now()),
            ('l10n_my_edi_retry_at', '=', False),
        ]])
        grouped_invoices = self.env["account.move"]._read_group(
            invoice_domain,
            groupby=["company_id", "l10n_my_edi_submission_uid"],
            aggregates=["id:recordset"],
            limit=MAX_SUBMISSION_UPDATE,
        )
        invoice_count = self.search_count(invoice_domain)  # Count the total amount of invoices to process.

        processed_invoices = 0
        for company, submission_uid, invoices in grouped_invoices:
            if not company.l10n_my_edi_proxy_user_id:
                continue

            error, status_fetch_result = self._l10n_my_get_submission_status(
                submission_uid, company.l10n_my_edi_proxy_user_id
            )
            if error:
                raise UserError(error)  # We do not expect errors here so raising is a correct solution.

            for invoice in invoices:
                invoice_result = status_fetch_result.get(invoice.l10n_my_edi_external_uuid)
                if not invoice_result:
                    continue

                # For valid invoices, we always want to update the try time; it's pointless to fetch too often.
                if invoice.l10n_my_edi_state == "valid" or invoice_result["status"] == "valid":
                    invoice.l10n_my_edi_retry_at = fields.Datetime.now() + datetime.timedelta(hours=1)

                if invoice_result["status"] == invoice.l10n_my_edi_state:
                    continue

                # If the state changed, we update the invoice with the new state and an eventual reason.
                invoice._l10n_my_edi_set_status(
                    state=invoice_result["status"],
                    message=_(
                        "This invoice has been %(status)s for reason: %(reason)s",
                        status=invoice_result["status"],
                        reason=invoice_result["reason"],
                    )
                    if invoice_result.get("reason")
                    else None,
                )
                if invoice.l10n_my_edi_state == "valid":
                    invoice._update_validation_fields(invoice_result)

            processed_invoices += len(invoices)
            # Commit if we can, in case an issue arises later.
            if self._can_commit():
                self.env['ir.cron']._notify_progress(done=processed_invoices, remaining=invoice_count - processed_invoices)
                self._cr.commit()

            time.sleep(0.3)  # There is a limit of how many calls we can do, so we pace ourselves
        self.env['ir.cron']._notify_progress(done=processed_invoices, remaining=invoice_count - processed_invoices)

    @api.model
    def _l10n_my_get_submission_status(self, submission_uid, proxy_user):
        """ Returns the status of all invoices in the submission.
        If there are too many and the submission is paginated, this will fetch each page with a waiting time of 1s per call.
        As a page can hold 100 invoice, it should not happen often.

        The proxy user is given as a param as this method can be called from the cron, in which case we can't rely on self.
        """
        # In case of errors, we return it alongside any results. We cannot raise as this is called from the send & print in some cases.
        error = ''
        # The api returns the result per document uuid, already correctly formated.
        # We do not need to format the data anymore for processing later but just to ensure we get complete data of the submission.
        result = proxy_user._l10n_my_edi_contact_proxy(
            endpoint='api/l10n_my_edi/1/get_submission_statuses',
            params={
                'submission_uid': submission_uid,
                'page': 1,
            }
        )

        if 'error' in result:
            error = self._l10n_my_edi_map_error(result['error'])
        else:
            if result['document_count'] <= 100:  # If so, we got all of it at once.
                result = result['statuses']
            else:
                # Otherwise we'll need to get the remaining invoices per batch of 100.
                for page in range(2, (result['document_count'] // 100) + 1):
                    time.sleep(1)  # To avoid any risks of throttling, we should wait a bit before continuing
                    page_result = proxy_user._l10n_my_edi_contact_proxy(
                        endpoint='api/l10n_my_edi/1/get_submission_statuses',
                        params={
                            'submission_uid': submission_uid,
                            'page': page,
                        }
                    )
                    result['statuses'].update(page_result['statuses'])
                result = result['statuses']
        return error, result

    def _l10n_my_edi_fetch_status(self):
        """ Action to fetch the status of a single invoice. """
        self.ensure_one()
        proxy_user = self._l10n_my_edi_ensure_proxy_user()

        # What to do with the given status is to be handled by the calling code.
        return proxy_user._l10n_my_edi_contact_proxy(
            endpoint='api/l10n_my_edi/1/get_status',
            params={
                'document_uuid': self.l10n_my_edi_external_uuid,
            },
        )

    def _update_validation_fields(self, validation_result):
        """ Update a few important fields in self based on the data received when an invoice gets to the 'valid' state. """
        self.ensure_one()
        # We receive a timezone_aware datetime, but it should always be in UTC.
        # Odoo expect a timezone unaware datetime in UTC, so we can safely remove the info without any more work needed.
        utc_tz_aware_datetime = dateutil.parser.isoparse(validation_result['valid_datetime'])
        self.l10n_my_edi_validation_time = utc_tz_aware_datetime.replace(tzinfo=None)

    # Other methods

    def _l10n_my_edi_uses_edi(self):
        """ Helper that returns true if the invoices company is using the Malaysian EDI.
        It does not mean that this specific invoice will use it, though.
        """
        self.ensure_one()
        proxy_user = self.company_id.l10n_my_edi_proxy_user_id
        return proxy_user and proxy_user.proxy_type == 'l10n_my_edi'

    def _l10n_my_edi_generate_invoice_xml(self):
        """ This edi's file is basically an ubl 2.1 file with some specificities. """
        self.ensure_one()
        return self.env['account.edi.xml.ubl_myinvois_my']._export_invoice(self)

    def _l10n_my_edi_ensure_proxy_user(self):
        # We need this fallback, as this method could be called by a cron.
        company = self.company_id or self.env.company

        proxy_user = company.l10n_my_edi_proxy_user_id
        if not proxy_user:
            raise UserError(_("Please register for the E-Invoicing service in the settings first."))

        return proxy_user

    def _l10n_my_edi_set_status(self, state, message=None):
        """ Small helper that changes the status, and log a message if a reason is provided. """
        if message:
            self._message_log_batch(bodies={move.id: message for move in self})

        self.l10n_my_edi_state = state

        # Once invalid, an invoice is not acceptable by the platform.
        # An invalid invoice will never be visible by a customer and should, from my understanding, be considered void.
        # In Odoo, the best way to represent that is by cancelling the invoice.
        if state in CANCELLED_STATES:
            self._l10n_my_edi_cancel_moves()

    def _l10n_my_edi_check_can_update_status(self):
        """ The document status can only be updated (for rejection, or cancellation) up to 72h after the validation time.
        After that, any update will be rejected by the platform, as you are expected to issue a debit/credit note.

        This helper will raise if the status cannot be updated.
        """
        self.ensure_one()
        if not self.l10n_my_edi_validation_time:
            return

        time_difference = datetime.datetime.now() - self.l10n_my_edi_validation_time
        if time_difference >= datetime.timedelta(days=3):
            raise UserError(_('It has been more than 72h since the invoice validation, you can no longer cancel it.\n'
                              'Instead, you should issue a debit or credit note.'))

    @api.model
    def _l10n_my_edi_map_error(self, error):
        """ This helper will take in an error code coming from the proxy, and return a translatable error message. """
        error_map = {
            # These errors should be returned when we send malformed request to the EDI, ... tldr; this should never happen unless we have bugs.
            'internal_server_error': _('Server error; If the problem persists, please contact the Odoo support.'),
            # The proxy user credentials are either incorrect, or Odoo does not have the permission to invoice on their behalf.
            'invalid_tin': _('Please make sure that your company TIN is correct, and that you gave Odoo sufficient permissions on the MyInvois platform.'),
            # The api rate limit has been reached. If this happens, we need to ask the user to wait. This is also handled proxy side to be safe
            'rate_limit_exceeded': _('The api request limit has been reached. Please wait until %(limit_reset_datetime)s to try again.',
                                     limit_reset_datetime=error.get('data')),  # Note, should be UTC. The TZ name is present in the formatted date.
            'hash_resubmitted': _('This document has already been submitted and was deemed invalid.\n'
                                  'Please correct the document based on the previous error, or wait before retrying.'),
            # This happens when the MyInvois TIN validator cannot validate the TIN of the user using the provided identification type and number.
            'document_tin_not_found': _('MyInvois could not match your TIN with the identification information you provided on the company.'),
            # This happens when the TIN of the supplier doesn't match with the TIN registered on the Proxy. Data contains the TIN.
            'document_tin_mismatch': _("The TIN number of the supplier in the invoices does not match with the one provided at the time of registering for the e-invoice service.\n"
                                       "If the TIN of the supplier's record changed after that, you will need to archive your EDI Proxy User and re-register.\n"
                                       "The TIN found in the document is %(tin_number)s",
                                       tin_number=error.get('data')),
            # This happens when a batch of invoices contains multiple different identifier for the supplier. Data contains the invoice.
            'multiple_documents_id': _('Multiple different supplier identification information were found in the invoices.\n'
                                       'If the company identification information changed, you may need to delete your invoice attachments and regenerate them.'),
            # Same as the previous error, but with the supplier TIN
            'multiple_documents_tin': _('Multiple different supplier TIN were found in the invoices.\n'
                                        'If the company TIN changed, you may need to delete your invoice attachments and regenerate them.'),
            # You cannot cancel an invoice that has been rejected or that is invalid
            'update_incorrect_state': _('You can only update the status of invoices in the valid state.'),
            'update_period_over': _('It has been more than 72h since the invoice validation, you can no longer update it.\n'
                                    'Instead, you should issue or request a debit or credit note.'),
            'update_active_documents': _('You cannot update this invoice, has it has been referenced by a debit or credit note.\n'
                                         'If you still want to update it, you must first update the debit/credit note.'),
            'update_forbidden': _('You do not have the permission to update this invoice.'),
            'search_date_invalid': _('The search params are invalid.'),  # Should never happen
        }

        if error.get('target'):
            # When validating a part of the invoice, they give random numerical codes with no explanation whatsoever.
            # So instead of trying to guess what they mean, we just give a generic "this is not valid" error and hope for the best.
            # For future bugfixer => To avoid issues as much as possible, please add additional checks in the UBL python file to avoid these.
            return _('An error occurred while validating the invoice: "%(property_name)s" is invalid.', property_name=error['target'])

        return error_map.get(error['reference'], _("An unexpected error has occurred."))

    def _l10n_my_edi_cancel_moves(self):
        """ Try to cancel the moves in self if allowed by the lock date. """
        for move in self:
            try:
                move._check_fiscal_lock_dates()
                move.line_ids._check_tax_lock_date()
                move.button_cancel()
            except UserError as e:
                move.with_context(no_new_invoice=True).message_post(
                    body=_(
                        'The invoice has been canceled on MyInvois, '
                        'But the cancellation in Odoo failed with error: %(error)s\n'
                        'Please resolve the problem manually, and then cancel the invoice.', error=e
                    )
                )

```

## File: models\account_move_send.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import time
from collections import defaultdict

from odoo import _, api, models


class AccountMoveSend(models.AbstractModel):
    _inherit = 'account.move.send'

    @api.model
    def _is_my_edi_applicable(self, move):
        return (move.is_invoice()
                and move.state == 'posted'
                and move.country_code == 'MY'
                and not move.l10n_my_edi_state
                and move.company_id.l10n_my_edi_proxy_user_id)

    def _get_all_extra_edis(self):
        # EXTENDS 'account'
        res = super()._get_all_extra_edis()
        res.update({'my_myinvois_send': {'label': _("Send to MyInvois"), 'is_applicable': self._is_my_edi_applicable}})
        return res

    # -------------------------------------------------------------------------
    # ALERTS
    # -------------------------------------------------------------------------

    def _get_alerts(self, moves, moves_data):
        # EXTENDS 'account'
        alerts = super()._get_alerts(moves, moves_data)
        if waiting_moves := moves.filtered(lambda m: m.l10n_my_edi_state == 'in_progress'):
            alerts['l10n_my_edi_warning_waiting_moves'] = {
                'message': _(
                    "The following invoice(s) are waiting for validation from MyInvois: %(move_name_list)s."
                    "Their status will be updated later on, or you can do it manually from the form view.",
                    move_name_list=', '.join(waiting_moves.mapped('name'))
                ),
                'action_text': _("View Invoice(s)"),
                'action': waiting_moves._get_records_action(name=_("Check Invoice(s)")),
            }
        return alerts

    # -------------------------------------------------------------------------
    # ATTACHMENTS
    # -------------------------------------------------------------------------

    @api.model
    def _get_invoice_extra_attachments(self, move):
        """ We are required to either:
            - Attach a QR code to the invoice PDF, that points to the e-invoice on the MyInvois platform
            - Attach the XML file we generated
        We will use to later for simplicity. It is unclear if the shared xml should be digitally signed or not.
        """
        # EXTENDS 'account'
        return (
            super()._get_invoice_extra_attachments(move)
            + move.l10n_my_edi_file_id
        )

    # -------------------------------------------------------------------------
    # SENDING METHODS
    # -------------------------------------------------------------------------

    @api.model
    def _l10n_my_edi_generate_myinvois_xml(self, invoice, invoice_data):
        need_file = (
            (invoice_data['invoice_edi_format'] == 'my_myinvois' and invoice.company_id.l10n_my_edi_proxy_user_id)
            or 'my_myinvois_send' in invoice_data['extra_edis']
        )
        # It should always be generated when sending.
        if need_file:
            # We don't pre-check the configuration, the ubl export will handle that part.
            xml_content, errors = invoice._l10n_my_edi_generate_invoice_xml()
            if errors:
                invoice_data['error'] = {
                    'error_title': _('Error when generating MyInvois file:'),
                    'errors': errors,
                }
            else:
                invoice_data['myinvois_attachments'] = [{
                    'name': f'{invoice.name.replace("/", "_")}_myinvois.xml',
                    'raw': xml_content,
                    'mimetype': 'application/xml',
                    'res_model': invoice._name,
                    'res_id': invoice.id,
                    'res_field': 'l10n_my_edi_file',  # Binary field
                }]

    @api.model
    def _hook_invoice_document_before_pdf_report_render(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._hook_invoice_document_before_pdf_report_render(invoice, invoice_data)
        self._l10n_my_edi_generate_myinvois_xml(invoice, invoice_data)

    @api.model
    def _call_web_service_before_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_before_invoice_pdf_render(invoices_data)

        xml_contents = defaultdict(list)
        moves = self.env['account.move']
        # This step is skipped if the move was sent, but not validated.
        for move, move_data in invoices_data.items():
            if 'my_myinvois_send' not in move_data['extra_edis']:
                continue

            moves |= move
            if 'myinvois_attachments' in move_data:
                xml_content = move_data['myinvois_attachments'][0]['raw'].decode('utf-8')
            # If the invoice was downloaded but not sent, the json file could already be there.
            elif move.l10n_my_edi_file:
                xml_content = base64.b64decode(move.l10n_my_edi_file).decode('utf-8')
            # If we don't have the file data and the file, we will regenerate it.
            else:
                self._l10n_my_edi_generate_myinvois_xml(move, move_data)
                if 'myinvois_attachments' not in move_data:
                    continue  # If an error occurred, it'll be in move_data['error'] so we can skip this invoice
                xml_content = move_data['myinvois_attachments'][0]['raw'].decode('utf-8')
            xml_contents[move] = xml_content

        if moves and xml_contents:
            errors = moves._l10n_my_edi_submit_documents(xml_contents)

            if errors:
                for move, move_data in invoices_data.items():
                    if move in errors:
                        move_data['error'] = {
                            'error_title': _('Error when sending the invoices to the E-invoicing service.'),
                            'errors': errors[move],
                        }

            # Whatever happened, we need to commit once at this point, because another api call is done later on
            # And in case of single invoice, a request error could raise => We would lose the uuid etc.
            if self._can_commit():
                self._cr.commit()

    def _call_web_service_after_invoice_pdf_render(self, invoices_data):
        """
        We need to get the submission status (valid, invalid) now for the flow to make sense.
        If we follow their documentation, the status should be available near instantly (in 2s of the submission) which
        means that it should be there in the time we get the submission response back and generate the invoice(s) pdf.

        We will try up to three time with a 2s delay to make it happen. It should cover most of the use cases, as long as
        there are no network issues/...

        If after three time the invoice still has not been processed, we will move on and leave the update to the
        scheduled action that fetches new incoming invoices and update statuses at the same time.
        """
        # EXTENDS 'account'
        super()._call_web_service_after_invoice_pdf_render(invoices_data)

        moves_in_progress = self.env['account.move']
        for move, move_data in invoices_data.items():
            if 'my_myinvois_send' not in move_data['extra_edis'] or move.l10n_my_edi_state != 'in_progress':
                continue

            moves_in_progress |= move

        # We want to ensure that we do not do anything more for moves which failed basic validations.
        if moves_in_progress:
            # This update can fail, but we don't consider that as a blocking error.
            # If the api request fails (timeout, validation not finished, ...) it'll be retried in the cron `ir_cron_myinvois_sync`.
            retry = 0
            errors, any_in_progress = moves_in_progress._l10n_my_edi_fetch_updated_statuses()
            while any_in_progress and retry < 2:
                time.sleep(1)  # We wait a second before retrying.
                errors, any_in_progress = moves_in_progress._l10n_my_edi_fetch_updated_statuses()
                retry += 1

            # While technically an in_progress status is not an error, it won't hurt much to display it as such.
            # The "error" message in this case should be clear enough.
            if errors:
                for move, move_data in invoices_data.items():
                    if move in errors:
                        move_data['error'] = {
                            'error_title': _('Error when fetching statuses from the E-invoicing service.'),
                            'errors': errors[move],
                        }

            # We commit again if possible, to ensure that the invoice status is set in the database in case of errors later.
            if self._can_commit():
                self._cr.commit()

    @api.model
    def _link_invoice_documents(self, invoices_data):
        # EXTENDS 'account'
        super()._link_invoice_documents(invoices_data)

        attachments_vals = []
        for invoice_data in invoices_data.values():
            attachments_vals.extend(invoice_data.get('myinvois_attachments', []))

        if attachments_vals:
            attachments = self.env['ir.attachment'].sudo().create(invoice_data.get('myinvois_attachments'))
            res_ids = attachments.mapped('res_id')
            self.env['account.move'].browse(res_ids).invalidate_recordset(fnames=['l10n_my_edi_file_id', 'l10n_my_edi_file'])

```

## File: models\account_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    # ------------------
    # Fields declaration
    # ------------------

    l10n_my_tax_type = fields.Selection(
        selection=[
            ('01', "Sales Tax"),
            ('02', "Service Tax"),
            ('03', "Tourism Tax"),
            ('04', "High-Value Goods Tax"),
            ('05', "Sales Tax on Low Value Goods"),
            ('06', "Not Applicable"),
            ('E', "Tax exemption (where applicable)"),
        ],
        string="Malaysian Tax Type",
        compute="_compute_l10n_my_tax_type",
        store=True,
        readonly=False,
    )

    # --------------------------------
    # Compute, inverse, search methods
    # --------------------------------

    @api.depends('amount', 'country_id', 'tax_scope')
    def _compute_l10n_my_tax_type(self):
        """ Compute default tax type based on a few factors. """
        for tax in self:
            if tax.country_id.code != 'MY':
                tax.l10n_my_tax_type = False
            else:
                if tax.amount == 0:
                    tax.l10n_my_tax_type = 'E'
                elif tax.tax_scope == 'consu':
                    tax.l10n_my_tax_type = '01'
                elif tax.tax_scope == 'service':
                    tax.l10n_my_tax_type = '02'
                else:
                    tax.l10n_my_tax_type = '06'

```

## File: models\l10n_my_edi_industry_classification.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class L10nMyEdiIndustryClassification(models.Model):
    """
    These codes are required by the API. They represent the industry classifications that are used in Malaysia.
    As defined in the list of MSIC codes allowed here: https://sdk.myinvois.hasil.gov.my/codes/msic-codes/

    Made a model as the list of codes would be too long for a selection field, yet it is easier to provide users with
    the list than expect them to find and then enter both name and code manually.
    """
    _name = 'l10n_my_edi.industry_classification'
    _description = "Malaysian Industry Classification"

    # ------------------
    # Fields declaration
    # ------------------

    name = fields.Char(required=True)
    code = fields.Char(required=True)

    # --------------------------------
    # Compute, inverse, search methods
    # --------------------------------

    @api.depends('code')
    def _compute_display_name(self):
        for classification in self:
            classification.display_name = f"{classification.code} {classification.name}"

```

## File: models\product_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

CLASSIFICATION_CODES_LIST = [
    ("001", "(001) Breastfeeding equipment "),
    ("002", "(002) Child care centres and kindergartens fees"),
    ("003", "(003) Computer, smartphone or tablet"),
    ("004", "(004) Consolidated e-Invoice "),
    (
        "005",
        "(005) Construction materials (as specified under Fourth Schedule of the Lembaga Pembangunan Industri Pembinaan Malaysia Act 1994)",
    ),
    ("006", "(006) Disbursement"),
    ("007", "(007) Donation"),
    ("008", "(008) -Commerce - e-Invoice to buyer / purchaser"),
    ("009", "(009) e-Commerce - Self-billed e-Invoice to seller, logistics, etc. "),
    ("010", "(010) Education fees"),
    ("011", "(011) Goods on consignment (Consignor)"),
    ("012", "(012) Goods on consignment (Consignee)"),
    ("013", "(013) Gym membership"),
    ("014", "(014) Insurance - Education and medical benefits"),
    ("015", "(015) Insurance - Takaful or life insurance"),
    ("016", "(016) Interest and financing expenses"),
    ("017", "(017) Internet subscription"),
    ("018", "(018) Land and building"),
    (
        "019",
        "(019) Medical examination for learning disabilities and early intervention or rehabilitation treatments of learning disabilities",
    ),
    ("020", "(020) Medical examination or vaccination expenses"),
    ("021", "(021) Medical expenses for serious diseases"),
    ("022", "(022) Others"),
    (
        "023",
        "(023) Petroleum operations (as defined in Petroleum (Income Tax) Act 1967)",
    ),
    ("024", "(024) Private retirement scheme or deferred annuity scheme"),
    ("025", "(025) Motor vehicle"),
    (
        "026",
        "(026) Subscription of books / journals / magazines / newspapers / other similar publications",
    ),
    ("027", "(027) Reimbursement"),
    ("028", "(028) Rental of motor vehicle"),
    (
        "029",
        "(029) EV charging facilities (Installation, rental, sale / purchase or subscription fees) ",
    ),
    ("030", "(030) Repair and maintenance"),
    ("031", "(031) Research and development"),
    ("032", "(032) Foreign income"),
    ("033", "(033) Self-billed - Betting and gaming"),
    ("034", "(034) Self-billed - Importation of goods"),
    ("035", "(035) Self-billed - Importation of services"),
    ("036", "(036) Self-billed - Others"),
    (
        "037",
        "(037) Self-billed - Monetary payment to agents, dealers or distributors",
    ),
    (
        "038",
        "(038) Fees related to sports equipment, facility rentals, competition registration, and training imposed by registered sports organizations under the Sports Development Act 1997",
    ),
    ("039", "(039) Supporting equipment for disabled person"),
    ("040", "(040) Voluntary contribution to approved provident fund "),
    ("041", "(041) Dental examination or treatment"),
    ("042", "(042) Fertility treatment"),
    (
        "043",
        "(043) Treatment and home care nursing, daycare centres and residential care centers",
    ),
    ("044", "(044) Vouchers, gift cards, loyalty points, etc"),
    (
        "045",
        "(045) Self-billed - Non-monetary payment to agents, dealers or distributors",
    ),
]


class ProductTemplate(models.Model):
    """
    These codes are required by the API. They represent the product classifications that are used in Malaysia.
    As defined in the list of codes allowed here: https://sdk.myinvois.hasil.gov.my/codes/classification-codes/
    """
    _inherit = "product.template"

    # ------------------
    # Fields declaration
    # ------------------

    l10n_my_edi_classification_code = fields.Selection(
        string="Malaysian classification code",
        selection=CLASSIFICATION_CODES_LIST,
    )

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    # ---------------
    # Default methods
    # ---------------

    def _default_l10n_my_edi_industrial_classification(self):
        return self.env.ref('l10n_my_edi.class_00000', raise_if_not_found=False)

    # ------------------
    # Fields declaration
    # ------------------

    l10n_my_edi_proxy_user_id = fields.Many2one(
        comodel_name="account_edi_proxy_client.user",
        compute="_compute_l10n_my_edi_proxy_user_id",
    )
    l10n_my_identification_type = fields.Selection(related='partner_id.l10n_my_identification_type', readonly=False)
    l10n_my_identification_number = fields.Char(related='partner_id.l10n_my_identification_number', readonly=False)
    l10n_my_identification_number_placeholder = fields.Char(compute="_compute_l10n_my_identification_number_placeholder")
    l10n_my_edi_industrial_classification = fields.Many2one(
        comodel_name='l10n_my_edi.industry_classification',
        string="Ind. Classification",
        default=_default_l10n_my_edi_industrial_classification,
    )
    l10n_my_edi_mode = fields.Selection(
        selection=[
            ('test', 'Pre-Production'),
            ('prod', 'Production'),
        ],
        # Nothing will happen until the user register, so it can be set by default.
        default="test",
    )
    # /!\ this was a planned feature that got scrapped due to API limitations. It may come back if their system provides better support for it.
    l10n_my_edi_default_import_journal_id = fields.Many2one(
        comodel_name="account.journal",
        domain="[('type', '=', 'purchase')]",
        string="Default import journal",
        help="The journal on which invoices imported from MyInvois will be booked. Leave empty to use the default purchase journal.",
    )

    # --------------------------------
    # Compute, inverse, search methods
    # --------------------------------

    @api.depends("account_edi_proxy_client_ids", 'l10n_my_edi_mode')
    def _compute_l10n_my_edi_proxy_user_id(self):
        """ Each company is expected to have at most one proxy user for malaysia for each mode.
        Thus, we can easily find said user.
        """
        for company in self:
            company.l10n_my_edi_proxy_user_id = company.account_edi_proxy_client_ids.filtered(
                lambda u: u.proxy_type == 'l10n_my_edi' and u.edi_mode == company.l10n_my_edi_mode
            )[:1]

    @api.depends('l10n_my_identification_type')
    def _compute_l10n_my_identification_number_placeholder(self):
        """ Computes a dynamic placeholder that depends on the selected type to help the user inputs their data.
        The placeholders have been taken from the MyInvois doc.
        """
        for company in self:
            placeholder = 'N/A'
            if company.l10n_my_identification_type == 'NRIC':
                placeholder = '830503-11-4923'
            elif company.l10n_my_identification_type == 'BRN':
                placeholder = '202201234565'
            elif company.l10n_my_identification_type == 'PASSPORT':
                placeholder = 'A00000000'
            elif company.l10n_my_identification_type == 'ARMY':
                placeholder = '830805-13-4983'
            company.l10n_my_identification_number_placeholder = placeholder

    # ----------------
    # Business methods
    # ----------------

    def _l10n_my_edi_create_proxy_user(self):
        """ This method will create a new proxy user for the current company based on the selected mode, if no users already exists. """
        self.ensure_one()
        if not self.l10n_my_edi_proxy_user_id:
            self.env['account_edi_proxy_client.user']._register_proxy_user(self, 'l10n_my_edi', self.l10n_my_edi_mode)

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    # ------------------
    # Fields declaration
    # ------------------

    l10n_my_edi_mode = fields.Selection(related="company_id.l10n_my_edi_mode", readonly=False)
    l10n_my_edi_default_import_journal_id = fields.Many2one(related="company_id.l10n_my_edi_default_import_journal_id", readonly=False)
    l10n_my_edi_proxy_user_id = fields.Many2one(related="company_id.l10n_my_edi_proxy_user_id")
    l10n_my_edi_company_vat = fields.Char(related="company_id.vat")
    l10n_my_accept_processing = fields.Boolean()

    # ----------------
    # Onchange methods
    # ----------------

    @api.onchange('l10n_my_edi_mode')
    def _onchange_l10n_my_edi_mode(self):
        """ This onchange is mostly here to improve usability by avoiding the need to save when changing the mode. """
        self.l10n_my_edi_proxy_user_id = self.company_id.account_edi_proxy_client_ids.filtered(
            lambda u: u.proxy_type == 'l10n_my_edi' and u.edi_mode == self.l10n_my_edi_mode
        )

    # --------------
    # Action methods
    # --------------

    def action_l10n_my_edi_allow_processing(self):
        """ We always expect the user to give his consent by pressing the button, in any mode, to enable the edi. """
        self.company_id._l10n_my_edi_create_proxy_user()

    def action_l10n_my_edi_unregister(self):
        """ Send a notification to the proxy to free the ID (vat) of the user, and archive the local proxy user.
        Useful if there has been a misconfiguration or the user wishes to use a new database/...
        """
        proxy_user = self.env.company.l10n_my_edi_proxy_user_id
        if not proxy_user:
            return

        # Start by notifying the proxy that we wish to deregister.
        result = proxy_user._l10n_my_edi_contact_proxy('api/l10n_my_edi/1/unregister', {})

        if not result.get('success'):
            # If we get a result, it should always be successful as we only archive. If for any reason it is not, we will raise an error.
            raise UserError(_("An unexpected error occurred while unregistering. Please try again later."))

        # If all goes well we can deactivate the local user.
        proxy_user.active = False

    def action_open_company_form(self):
        """ This will be used to ease the configuration by allowing to quickly access the company. """
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'res_id': self.env.company.id,
            'res_model': 'res.company',
            'target': 'new',
            'view_mode': 'form',
        }

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class ResPartner(models.Model):
    _inherit = 'res.partner'

    # ------------------
    # Fields declaration
    # ------------------

    l10n_my_tin_validation_state = fields.Selection(
        selection=[
            ('valid', 'Valid'),
            ('invalid', 'Invalid'),
        ],
        string='Tin Validation State',
        help="Technical field, hold the result of TIN validation using MyInvois API.\n"
             "It is non blocking, and will simply help ensure that the customer of an invoice is valid to avoid submission errors.",
        compute='_compute_l10n_my_tin_validation_state',
        readonly=False,
        store=True,
        export_string_translation=False,
    )
    l10n_my_edi_display_tin_warning = fields.Boolean(
        compute='_compute_l10n_my_edi_display_tin_warning',
    )

    l10n_my_identification_type = fields.Selection(
        string="ID Type",
        selection=[
            ('NRIC', 'MyKad/MyTentera/MyPR/MyKAS'),
            ('BRN', 'Business Registration Number'),
            ('PASSPORT', 'Passport'),
            ('ARMY', 'Army'),
        ],
        default="BRN",
        help="The identification type and number used by the MyTax/MyInvois system to identify the user.\nNote: For MyPR and MyKAS to use NRIC scheme",
    )
    l10n_my_identification_number = fields.Char(string="ID Number")
    l10n_my_identification_number_placeholder = fields.Char(compute="_compute_l10n_my_identification_number_placeholder")

    # --------------------------------
    # Compute, inverse, search methods
    # --------------------------------

    @api.depends('l10n_my_identification_type', 'l10n_my_identification_number', 'vat')
    def _compute_l10n_my_tin_validation_state(self):
        """ The three @depends are used for the validation. If they change, we will invalidate it and expect the user to revalidate. """
        self.l10n_my_tin_validation_state = False

    @api.depends_context('company', 'l10n_my_identification_number')
    def _compute_l10n_my_edi_display_tin_warning(self):
        """ We want to display the tin warning for companies registered to use MyInvois. """
        # We need to sudo here, as all users having access to partners may not have the rights to access the proxy users.
        proxy_user = self.env.company.sudo().l10n_my_edi_proxy_user_id
        is_edi_used = proxy_user and proxy_user.proxy_type == 'l10n_my_edi'
        for partner in self:
            # Users with no business number can't be validated using the api
            partner.l10n_my_edi_display_tin_warning = is_edi_used and partner.l10n_my_identification_number

    @api.depends('l10n_my_identification_type')
    def _compute_l10n_my_identification_number_placeholder(self):
        """ Computes a dynamic placeholder that depends on the selected type to help the user inputs their data.
        The placeholders have been taken from the MyInvois doc.
        """
        for partner in self:
            placeholder = 'N/A'
            if partner.l10n_my_identification_type == 'NRIC':
                placeholder = '830503-11-4923'
            elif partner.l10n_my_identification_type == 'BRN':
                placeholder = '202201234565'
            elif partner.l10n_my_identification_type == 'PASSPORT':
                placeholder = 'A00000000'
            elif partner.l10n_my_identification_type == 'ARMY':
                placeholder = '830805-13-4983'
            partner.l10n_my_identification_number_placeholder = placeholder

    # --------------
    # Action methods
    # --------------

    def action_validate_tin(self):
        """ Calling this action will reach our EDI proxy in order to validate the TIN against the provided identification information. """
        self.ensure_one()
        if not self._l10n_my_edi_get_tin_for_myinvois() or not self.l10n_my_identification_type or not self.l10n_my_identification_number:
            raise UserError(_('In order to validate the TIN, you must provide the Identification type and number.'))

        # Sudo to allow a user without access to the proxy user to validate the ID if needed.
        proxy_user = self.env.company.sudo().l10n_my_edi_proxy_user_id
        if not proxy_user:
            raise UserError(_("Please register for the E-Invoicing service in the settings first."))

        response = proxy_user._l10n_my_edi_contact_proxy('api/l10n_my_edi/1/validate_tin', params={
            'identification_values': {
                'tin': self._l10n_my_edi_get_tin_for_myinvois(),
                'id_type': self.l10n_my_identification_type,
                'id_val': self.l10n_my_identification_number,
            }
        })

        if 'error' in response:
            ref = response['error']['reference']
            # No need to rollback, we don't want to be blocking on that.
            if ref == 'document_tin_not_found':
                self._message_log(body=_('MyInvois was not able to match the TIN with the provided identification number.\nThis may happen when using generic TIN and will not prevent you from invoicing.'))
                self.l10n_my_tin_validation_state = 'invalid'
            else:
                self._message_log(body=_('An unexpected error occurred while validating the TIN. Please try again later.'))
        else:
            self.l10n_my_tin_validation_state = 'valid' if response.get('success') else 'invalid'

    def _l10n_my_edi_get_tin_for_myinvois(self):
        """ Helper to return the VAT number relevant to the situation. """
        self.ensure_one()
        return self.vat

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['l10n_my_identification_type', 'l10n_my_identification_number']

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_edi_proxy_user
from . import account_edi_xml_ubl_my
from . import account_move
from . import account_move_send
from . import account_tax
from . import l10n_my_edi_industry_classification
from . import product_template
from . import res_company
from . import res_config_settings
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
access_l10n_my_edi_industry_classification_readonly,l10n_my_edi.industry_classification,model_l10n_my_edi_industry_classification,account.group_account_readonly,1,0,0,0
access_l10n_my_edi_industry_classification_invoice,l10n_my_edi.industry_classification,model_l10n_my_edi_industry_classification,account.group_account_invoice,1,0,0,0
access_l10n_my_edi_industry_classification_manager,l10n_my_edi.industry_classification,model_l10n_my_edi_industry_classification,account.group_account_manager,1,1,1,1
access_l10n_my_edi_document_status_update_readonly,l10n_my_edi.document.status.update,model_l10n_my_edi_document_status_update,account.group_account_readonly,1,0,0,0
access_l10n_my_edi_document_status_update_invoice,l10n_my_edi.document.status.update,model_l10n_my_edi_document_status_update,account.group_account_invoice,1,0,0,0
access_l10n_my_edi_document_status_update_manager,l10n_my_edi.document.status.update,model_l10n_my_edi_document_status_update,account.group_account_manager,1,1,1,1

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_move_form_inherit_l10n_my_myinvois" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n_my_myinvois</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//widget[@name='web_ribbon']" position="before">
                <widget name="web_ribbon" title="Processing" bg_color="text-bg-info" invisible="l10n_my_edi_state != 'in_progress'"/>
                <widget name="web_ribbon" title="Rejected" bg_color="text-bg-warning" invisible="l10n_my_edi_state != 'rejected'"/>
            </xpath>
            <xpath expr="//header" position="inside">
                <!-- The invalid/cancelled states should be final. -->
                <!-- This button is not needed on rejected invoices as it expects an action from the user -->
                <button name="action_l10n_my_edi_update_status" string="Update MyInvois Status" type="object"
                        groups="account.group_account_invoice"
                        invisible="not l10n_my_edi_state or (l10n_my_edi_state not in ['in_progress', 'valid']) or (l10n_my_edi_state == 'rejected' and move_type not in ('out_invoice', 'out_refund'))"/>
                <button name="action_l10n_my_edi_reject_bill" string="Reject" type="object"
                        groups="account.group_account_invoice"
                        invisible="l10n_my_edi_state != 'valid' or move_type not in ('in_invoice', 'in_refund')"/>
            </xpath>
            <!-- There are quite a few fields, and with other modules adding some too it would be too messy to not put them in a tab. -->
            <xpath expr="//notebook" position="inside">
                <page string="MyInvois" invisible="country_code != 'MY'">
                    <group>
                        <group>
                            <!-- Only displayed if the invoice contains a tax of type Exempt -->
                            <field name="l10n_my_edi_display_tax_exemption_reason" invisible="1"/>
                            <field name="l10n_my_edi_exemption_reason" invisible="not l10n_my_edi_display_tax_exemption_reason" readonly="l10n_my_edi_state != False"/>
                            <field name="l10n_my_edi_custom_form_reference" readonly="l10n_my_edi_state != False"/>
                        </group>
                        <group>
                            <!-- a "not sent" placeholder would be great, but they don't work in readonly -->
                            <field name="l10n_my_edi_state" invisible="not l10n_my_edi_state"/>
                            <field name="l10n_my_edi_submission_uid" invisible="not l10n_my_edi_submission_uid"/>
                            <field name="l10n_my_edi_external_uuid" invisible="not l10n_my_edi_external_uuid"/>
                            <field name="l10n_my_edi_validation_time" invisible="not l10n_my_edi_validation_time"/>
                        </group>
                    </group>
                </page>
            </xpath>
        </field>
    </record>

    <record id="view_invoice_list_inherit_l10n_my_myinvois" model="ir.ui.view">
        <field name="name">account.move.list.inherit.l10n_my_myinvois</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_invoice_tree" />
        <field name="arch" type="xml">
            <field name="status_in_payment" position="before">
                <field name="l10n_my_edi_state" optional="hide"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\account_tax_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_tax_form_inherit_l10n_my_myinvois" model="ir.ui.view">
        <field name="name">account.tax.form.inherit.l10n_my_myinvois</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='active']" position="after">
                <field name="l10n_my_tax_type" invisible="country_code != 'MY'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\l10n_my_edi_industrial_classification_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Just there for the search more -->
        <record id="view_classification_list" model="ir.ui.view">
            <field name="name">l10n_my_edi.industry_classification.list</field>
            <field name="model">l10n_my_edi.industry_classification</field>
            <field name="arch" type="xml">
                <list>
                    <field name="code"/>
                    <field name="name"/>
                </list>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\product_template_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="product_template_form_view" model="ir.ui.view">
            <field name="name">product.template.form.inherit</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_form_view"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='group_general']" position="inside">
                    <field name="l10n_my_edi_classification_code" invisible="'MY' not in fiscal_country_codes"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_company_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_company_form_inherit_l10n_my_myinvois" model="ir.ui.view">
        <field name="name">res.company.form.inherit.l10n_my_myinvois</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='general_info']" position="inside">
                <group invisible="country_code != 'MY'">
                    <group string="E-Invoicing" colspan="2">
                        <label for="l10n_my_identification_type" string="Identification"/>
                        <div class="d-flex gap-2 w-50">
                            <field name="l10n_my_identification_type"/>
                            <span class="d-flex gap-2 w-50">
                                <field name="l10n_my_identification_number_placeholder" invisible="1"/> <!-- Needed for the placeholder widget -->
                                <field name="l10n_my_identification_number" options="{'placeholder_field': 'l10n_my_identification_number_placeholder'}"/>
                            </span>
                        </div>
                        <field class="w-50" name="l10n_my_edi_industrial_classification"/>
                    </group>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.proxy.user</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@id='account_vendor_bills']" position="after">
                <block title="Malaysian Electronic Invoicing" id='malaysian_edi' invisible="country_code != 'MY'">
                    <setting class="col-lg-12" string="MyInvois mode" company_dependent="1">
                        <field name="l10n_my_edi_proxy_user_id" invisible="1"/>
                        <div class="content-group">
                            <field name="l10n_my_edi_mode" widget="radio"/>
                        </div>
                        <div class="mt8 content-group col-9" invisible="l10n_my_edi_mode == 'prod'">
                            <span invisible="l10n_my_edi_mode != 'test'">• In Pre-Production mode Odoo will send the invoices to a non-production service.</span>
                        </div>
                        <!-- No need to show the button if there is already a user for the selected mode and company. -->
                        <div class="mt8 content-group" invisible="not l10n_my_edi_mode or l10n_my_edi_proxy_user_id">
                            <span><field name="l10n_my_accept_processing"/> I accept that Odoo will process my e-invoices.</span>
                            <div class="text-muted" invisible="not l10n_my_edi_company_vat">
                                The TIN
                                <a name="action_open_company_form" type="object" role="button" class="btn-link">
                                    <field name="l10n_my_edi_company_vat" class="oe_inline"/>
                                </a>
                                will be used when sending e-invoices.<br/>
                            </div>
                            <button name="action_l10n_my_edi_allow_processing" type="object" string="Register" class="btn btn-secondary mt-1" noSaveDialog="true" invisible="not l10n_my_accept_processing"/>
                        </div>
                        <div class="mt8 text-success" invisible="not l10n_my_edi_proxy_user_id">
                            <div>The electronic invoicing service is in use.</div>
                            <button name="action_l10n_my_edi_unregister" type="object" string="Unregister" class="btn btn-secondary mt-1" noSaveDialog="true"/>
                        </div>
                    </setting>
                </block>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_partner_form_inherit_l10n_my_myinvois" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_my_myinvois</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <group name="container_row_2" position="inside">
                <field name="l10n_my_tin_validation_state" invisible="1"/>
                <field name="l10n_my_edi_display_tin_warning" invisible="1"/>
                <!-- Foreigner with a tax number registered in Malaysia could be customer of an e-invoice. -->
                <group name="l10n_my_edi" string="MyInvois Information" invisible="'MY' not in fiscal_country_codes">
                    <group colspan="2">
                        <label for="l10n_my_identification_type" string="Identification"/>
                        <div class="d-flex gap-2">
                            <field name="l10n_my_identification_type"  readonly="parent_id"/>
                            <span class="d-flex gap-2 w-100">
                                <field name="l10n_my_identification_number_placeholder" invisible="1"/> <!-- Needed for the placeholder widget -->
                                <field name="l10n_my_identification_number" options="{'placeholder_field': 'l10n_my_identification_number_placeholder'}" readonly="parent_id"/>
                                <button class="oe_link oe_inline p-0" type="object" name="action_validate_tin" invisible="not l10n_my_edi_display_tin_warning or l10n_my_tin_validation_state or parent_id">Validate</button>
                                <span class="text-success fa fa-check" title="Validation Successful" invisible="not l10n_my_edi_display_tin_warning or l10n_my_tin_validation_state != 'valid'"/>
                                <span class="text-danger fa fa-close" title="Validation Failed" invisible="not l10n_my_edi_display_tin_warning or l10n_my_tin_validation_state != 'invalid'"/>
                            </span>
                        </div>
                    </group>
                </group>
            </group>
        </field>
    </record>
</odoo>

```

## File: wizard\l10n_my_edi_status_update_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models
from odoo.exceptions import UserError


class L10nMyEdiStatusUpdateWizard(models.TransientModel):
    _name = 'l10n_my_edi.document.status.update'
    _description = 'Document Status Update Wizard'

    invoice_id = fields.Many2one(
        comodel_name='account.move',
        string='Document To Update',
        required=True,
        readonly=True,
    )
    reason = fields.Char(
        help='Reason for cancelling the document.',
        required=True,
    )
    new_status = fields.Char(
        help='New status to set on the document.',
        required=True,
        readonly=True,
    )

    def button_request_update(self):
        self.ensure_one()
        if not self.reason.strip():
            raise UserError(_('You must provide a reason for updating the document.'))

        self.invoice_id._l10n_my_edi_update_document(status=self.new_status, reason=self.reason)

```

## File: wizard\l10n_my_edi_status_update_wizard.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_my_edi_document_status_update_form" model="ir.ui.view">
        <field name="name">l10n_my_edi.document.status.update.form</field>
        <field name="model">l10n_my_edi.document.status.update</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="invoice_id"/>
                    <field name="new_status"/>
                    <field name="reason"/>
                </group>
                <footer>
                    <button string="Update Invoice" name="button_request_update" type="object" default_focus="1" class="btn-primary"/>
                    <button string="Close" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import l10n_my_edi_status_update_wizard

```

