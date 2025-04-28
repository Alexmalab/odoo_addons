# Odoo Module: account_sequence

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 CCI Connect asbl (http://www.cciconnect.be) All Rights Reserved.
#                       Philmer <philmer@cciconnect.be>

{
    'name': 'Accounting Sequence',
    'version': '1.0',
    'category': 'Hidden',
    'description': "Change the way `sequence.mixin` works to reduce concurrency errors",
    'depends': ['account'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
from odoo import models
from odoo.tools import index_exists


class AccountMove(models.Model):
    _inherit = 'account.move'

    _sql_constraints = [(
        'unique_name', "", "Another entry with the same name already exists.",
    )]

    def _auto_init(self):
        super()._auto_init()
        if not index_exists(self.env.cr, 'account_move_unique_name'):
            # Make all values of `name` different (naming them `name (1)`, `name (2)`...) so that we can add the following UNIQUE INDEX
            self.env.cr.execute("""
                WITH duplicated_sequence AS (
                    SELECT name, journal_id, state
                      FROM account_move
                     WHERE state = 'posted'
                       AND name != '/'
                  GROUP BY journal_id, name, state
                    HAVING COUNT(*) > 1
                ),
                to_update AS (
                    SELECT move.id,
                           move.name,
                           move.journal_id,
                           move.state,
                           move.date,
                           row_number() OVER(PARTITION BY move.name, move.journal_id ORDER BY move.name, move.journal_id, move.date) AS row_seq
                      FROM duplicated_sequence
                      JOIN account_move move ON move.name = duplicated_sequence.name
                                            AND move.journal_id = duplicated_sequence.journal_id
                                            AND move.state = duplicated_sequence.state
                ),
                new_vals AS (
                    SELECT id,
                           name || ' (' || (row_seq-1)::text || ')' AS name
                      FROM to_update
                     WHERE row_seq > 1
                )
                UPDATE account_move
                   SET name = new_vals.name
                  FROM new_vals
                 WHERE account_move.id = new_vals.id;
            """)
            self.env.cr.execute("""
                CREATE UNIQUE INDEX account_move_unique_name
                ON account_move(name, journal_id) WHERE (state = 'posted' AND name != '/');
            """)

    def _check_unique_sequence_number(self):
        return

```

## File: models\sequence_mixin.py

```python
from psycopg2 import DatabaseError

from odoo import models
from odoo.tools import mute_logger


class SequenceMixin(models.AbstractModel):
    _inherit = 'sequence.mixin'

    def _get_last_sequence(self, relaxed=False, with_prefix=None, lock=True):
        return super()._get_last_sequence(relaxed, with_prefix, False)

    def _set_next_sequence(self):
        # OVERRIDE
        self.ensure_one()
        last_sequence = self._get_last_sequence()
        new = not last_sequence
        if new:
            last_sequence = self._get_last_sequence(relaxed=True) or self._get_starting_sequence()

        format_string, format_values = self._get_sequence_format_param(last_sequence)
        sequence_number_reset = self._deduce_sequence_number_reset(last_sequence)
        if new:
            date_start, date_end = self._get_sequence_date_range(sequence_number_reset)
            format_values['seq'] = 0
            format_values['year'] = self._truncate_year_to_length(date_start.year, format_values['year_length'])
            format_values['month'] = date_start.month
            format_values['year_end'] = self._truncate_year_to_length(date_end.year, format_values['year_end_length'])
        # before flushing inside the savepoint (which may be rolled back!), make sure everything
        # is already flushed, otherwise we could lose non-sequence fields values, as the ORM believes
        # them to be flushed.
        self.flush_recordset()
        # because we are flushing, and because the business code might be flushing elsewhere (i.e. to
        # validate constraints), the fields depending on the sequence field might be protected by the
        # ORM. This is not desired, so we already reset them here.
        registry = self.env.registry
        triggers = registry._field_triggers[self._fields[self._sequence_field]]
        for inverse_field, triggered_fields in triggers.items():
            for triggered_field in triggered_fields:
                if not triggered_field.store or not triggered_field.compute:
                    continue
                for field in registry.field_inverses[inverse_field[0]] if inverse_field else [None]:
                    self.env.add_to_compute(triggered_field, self[field.name] if field else self)
        while True:
            format_values['seq'] = format_values['seq'] + 1
            sequence = format_string.format(**format_values)
            try:
                with self.env.cr.savepoint(flush=False), mute_logger('odoo.sql_db'):
                    self[self._sequence_field] = sequence
                    self.flush_recordset([self._sequence_field])
                    break
            except DatabaseError as e:
                # 23P01 ExclusionViolation
                # 23505 UniqueViolation
                if e.pgcode not in ('23P01', '23505'):
                    raise e
        self._compute_split_sequence()
        self.flush_recordset(['sequence_prefix', 'sequence_number'])

```

## File: models\__init__.py

```python
from . import account_move
from . import sequence_mixin

```

