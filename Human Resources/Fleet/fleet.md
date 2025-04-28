# Odoo Module: fleet

Category: Human Resources/Fleet

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name' : 'Fleet',
    'version' : '0.1',
    'sequence': 185,
    'category': 'Human Resources/Fleet',
    'website' : 'https://www.odoo.com/app/fleet',
    'summary' : 'Manage your fleet and track car costs',
    'description' : """
Vehicle, leasing, insurances, cost
==================================
With this module, Odoo helps you managing all your vehicles, the
contracts associated to those vehicle as well as services, costs
and many other features necessary to the management of your fleet
of vehicle(s)

Main Features
-------------
* Add vehicles to your fleet
* Manage contracts for vehicles
* Reminder when a contract reach its expiration date
* Add services, odometer values for all vehicles
* Show all costs associated to a vehicle or to a type of service
* Analysis graph for costs
""",
    'depends': [
        'base',
        'mail',
    ],
    'data': [
        'security/fleet_security.xml',
        'security/ir.model.access.csv',
        'views/fleet_vehicle_model_views.xml',
        'views/fleet_vehicle_views.xml',
        'views/fleet_vehicle_cost_views.xml',
        'views/fleet_board_view.xml',
        'views/mail_activity_views.xml',
        'views/res_config_settings_views.xml',
        'data/fleet_cars_data.xml',
        'data/fleet_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_activity_type_data.xml',
    ],

    'demo': ['data/fleet_demo.xml'],

    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'fleet/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\fleet_cars_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
      <record id="brand_abarth" model="fleet.vehicle.model.brand">
      	<field name="name">Abarth</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_abarth-image.png"/>
      </record>
      <record id="brand_acura" model="fleet.vehicle.model.brand">
      	<field name="name">Acura</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_acura-image.png"/>
      </record>
      <record id="brand_alfa" model="fleet.vehicle.model.brand">
      	<field name="name">Alfa</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_alfa-image.png"/>
      </record>
      <record id="brand_audi" model="fleet.vehicle.model.brand">
      	<field name="name">Audi</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_audi-image.png"/>
      </record>
      <record id="brand_austin" model="fleet.vehicle.model.brand">
      	<field name="name">Austin</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_austin-image.png"/>
      </record>
      <record id="brand_bentley" model="fleet.vehicle.model.brand">
      	<field name="name">Bentley</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_bentley-image.png"/>
      </record>
      <record id="brand_bmw" model="fleet.vehicle.model.brand">
      	<field name="name">Bmw</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_bmw-image.png"/>
      </record>
      <record id="brand_bugatti" model="fleet.vehicle.model.brand">
      	<field name="name">Bugatti</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_bugatti-image.png"/>
      </record>
      <record id="brand_buick" model="fleet.vehicle.model.brand">
      	<field name="name">Buick</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_buick-image.png"/>
      </record>
      <record id="brand_byd" model="fleet.vehicle.model.brand">
      	<field name="name">Byd</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_byd-image.png"/>
      </record>
      <record id="brand_cadillac" model="fleet.vehicle.model.brand">
      	<field name="name">Cadillac</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_cadillac-image.png"/>
      </record>
      <record id="brand_chevrolet" model="fleet.vehicle.model.brand">
      	<field name="name">Chevrolet</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_chevrolet-image.png"/>
      </record>
      <record id="brand_chrysler" model="fleet.vehicle.model.brand">
      	<field name="name">Chrysler</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_chrysler-image.png"/>
      </record>
      <record id="brand_citroen" model="fleet.vehicle.model.brand">
      	<field name="name">Citroen</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_citroen-image.png"/>
      </record>
      <record id="brand_corre_la_licorne" model="fleet.vehicle.model.brand">
      	<field name="name">Corre La Licorne</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_corre-la-licorne-image.png"/>
      </record>
      <record id="brand_daewoo" model="fleet.vehicle.model.brand">
      	<field name="name">Daewoo</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_daewoo-image.png"/>
      </record>
      <record id="brand_dodge" model="fleet.vehicle.model.brand">
      	<field name="name">Dodge</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_dodge-image.png"/>
      </record>
      <record id="brand_ferrari" model="fleet.vehicle.model.brand">
      	<field name="name">Ferrari</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_ferrari-image.png"/>
      </record>
      <record id="brand_fiat" model="fleet.vehicle.model.brand">
      	<field name="name">Fiat</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_fiat-image.png"/>
      </record>
      <record id="brand_ford" model="fleet.vehicle.model.brand">
      	<field name="name">Ford</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_ford-image.png"/>
      </record>
      <record id="brand_holden" model="fleet.vehicle.model.brand">
      	<field name="name">Holden</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_holden-image.png"/>
      </record>
      <record id="brand_honda" model="fleet.vehicle.model.brand">
      	<field name="name">Honda</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_honda-image.png"/>
      </record>
      <record id="brand_hyundai" model="fleet.vehicle.model.brand">
      	<field name="name">Hyundai</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_hyundai-image.png"/>
      </record>
      <record id="brand_infiniti" model="fleet.vehicle.model.brand">
      	<field name="name">Infiniti</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_infiniti-image.png"/>
      </record>
      <record id="brand_isuzu" model="fleet.vehicle.model.brand">
      	<field name="name">Isuzu</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_isuzu-image.png"/>
      </record>
      <record id="brand_jaguar" model="fleet.vehicle.model.brand">
      	<field name="name">Jaguar</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_jaguar-image.png"/>
      </record>
      <record id="brand_jeep" model="fleet.vehicle.model.brand">
      	<field name="name">Jeep</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_jeep-image.png"/>
      </record>
      <record id="brand_kia" model="fleet.vehicle.model.brand">
      	<field name="name">Kia</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_kia-image.png"/>
      </record>
      <record id="brand_koenigsegg" model="fleet.vehicle.model.brand">
      	<field name="name">Koenigsegg</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_koenigsegg-image.png"/>
      </record>
      <record id="brand_lagonda" model="fleet.vehicle.model.brand">
      	<field name="name">Lagonda</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_lagonda-image.png"/>
      </record>
      <record id="brand_lamborghini" model="fleet.vehicle.model.brand">
      	<field name="name">Lamborghini</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_lamborghini-image.png"/>
      </record>
      <record id="brand_lancia" model="fleet.vehicle.model.brand">
      	<field name="name">Lancia</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_lancia-image.png"/>
      </record>
      <record id="brand_land_rover" model="fleet.vehicle.model.brand">
      	<field name="name">Land Rover</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_land-rover-image.png"/>
      </record>
      <record id="brand_lexus" model="fleet.vehicle.model.brand">
      	<field name="name">Lexus</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_lexus-image.png"/>
      </record>
      <record id="brand_lincoln" model="fleet.vehicle.model.brand">
      	<field name="name">Lincoln</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_lincoln-image.png"/>
      </record>
      <record id="brand_lotus" model="fleet.vehicle.model.brand">
      	<field name="name">Lotus</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_lotus-image.png"/>
      </record>
      <record id="brand_maserati" model="fleet.vehicle.model.brand">
      	<field name="name">Maserati</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_maserati-image.png"/>
      </record>
      <record id="brand_maybach" model="fleet.vehicle.model.brand">
      	<field name="name">Maybach</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_maybach-image.png"/>
      </record>
      <record id="brand_mazda" model="fleet.vehicle.model.brand">
      	<field name="name">Mazda</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_mazda-image.png"/>
      </record>
      <record id="brand_mercedes" model="fleet.vehicle.model.brand">
      	<field name="name">Mercedes</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_mercedes-image.png"/>
      </record>
      <record id="brand_mg" model="fleet.vehicle.model.brand">
      	<field name="name">Mg</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_mg-image.png"/>
      </record>
      <record id="brand_mini" model="fleet.vehicle.model.brand">
      	<field name="name">Mini</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_mini-image.png"/>
      </record>
      <record id="brand_mitsubishi" model="fleet.vehicle.model.brand">
      	<field name="name">Mitsubishi</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_mitsubishi-image.png"/>
      </record>
      <record id="brand_morgan" model="fleet.vehicle.model.brand">
      	<field name="name">Morgan</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_morgan-image.png"/>
      </record>
      <record id="brand_nissan" model="fleet.vehicle.model.brand">
      	<field name="name">Nissan</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_nissan-image.png"/>
      </record>
      <record id="brand_oldsmobile" model="fleet.vehicle.model.brand">
      	<field name="name">Oldsmobile</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_oldsmobile-image.png"/>
      </record>
      <record id="brand_opel" model="fleet.vehicle.model.brand">
      	<field name="name">Opel</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_opel-image.png"/>
      </record>
      <record id="brand_peugeot" model="fleet.vehicle.model.brand">
      	<field name="name">Peugeot</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_peugeot-image.png"/>
      </record>
      <record id="brand_pontiac" model="fleet.vehicle.model.brand">
      	<field name="name">Pontiac</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_pontiac-image.png"/>
      </record>
      <record id="brand_porsche" model="fleet.vehicle.model.brand">
      	<field name="name">Porsche</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_porsche-image.png"/>
      </record>
      <record id="brand_rambler" model="fleet.vehicle.model.brand">
      	<field name="name">Rambler</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_rambler-image.png"/>
      </record>
      <record id="brand_renault" model="fleet.vehicle.model.brand">
      	<field name="name">Renault</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_renault-image.png"/>
      </record>
      <record id="brand_rolls-royce" model="fleet.vehicle.model.brand">
      	<field name="name">Rolls-Royce</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_rolls-royce-image.png"/>
      </record>
      <record id="brand_saab" model="fleet.vehicle.model.brand">
      	<field name="name">Saab</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_saab-image.png"/>
      </record>
      <record id="brand_scion" model="fleet.vehicle.model.brand">
      	<field name="name">Scion</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_scion-image.png"/>
      </record>
      <record id="brand_skoda" model="fleet.vehicle.model.brand">
      	<field name="name">Skoda</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_skoda-image.png"/>
      </record>
      <record id="brand_smart" model="fleet.vehicle.model.brand">
      	<field name="name">Smart</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_smart-image.png"/>
      </record>
      <record id="brand_steyr" model="fleet.vehicle.model.brand">
      	<field name="name">Steyr</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_steyr-image.png"/>
      </record>
      <record id="brand_subaru" model="fleet.vehicle.model.brand">
      	<field name="name">Subaru</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_subaru-image.png"/>
      </record>
      <record id="brand_tesla_motors" model="fleet.vehicle.model.brand">
      	<field name="name">Tesla Motors</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_tesla-motors-image.png"/>
      </record>
      <record id="brand_toyota" model="fleet.vehicle.model.brand">
      	<field name="name">Toyota</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_toyota-image.png"/>
      </record>
      <record id="brand_trabant" model="fleet.vehicle.model.brand">
      	<field name="name">Trabant</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_trabant-image.png"/>
      </record>
      <record id="brand_volkswagen" model="fleet.vehicle.model.brand">
      	<field name="name">Volkswagen</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_volkswagen-image.png"/>
      </record>
      <record id="brand_volvo" model="fleet.vehicle.model.brand">
      	<field name="name">Volvo</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_volvo-image.png"/>
      </record>
      <record id="brand_willys" model="fleet.vehicle.model.brand">
      	<field name="name">Willys</field>
      	<field name="image_128" type="base64" file="fleet/static/img/brand_willys-image.png"/>
      </record>
      <record id="brand_suzuki" model="fleet.vehicle.model.brand">
        <field name="name">Suzuki</field>
        <field name="image_128" type="base64" file="fleet/static/img/brand_suzuki-image.png"/>
      </record>
      <record id="model_corsa" model="fleet.vehicle.model">
          <field name="name">Corsa</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_astra" model="fleet.vehicle.model">
          <field name="name">Astra</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_agila" model="fleet.vehicle.model">
          <field name="name">Agila</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_combotour" model="fleet.vehicle.model">
          <field name="name">Combo Tour</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_meriva" model="fleet.vehicle.model">
          <field name="name">Meriva</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_astragtc" model="fleet.vehicle.model">
          <field name="name">AstraGTC</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_zafira" model="fleet.vehicle.model">
          <field name="name">Zafira</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_zafiratourer" model="fleet.vehicle.model">
          <field name="name">Zafira Tourer</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_insignia" model="fleet.vehicle.model">
          <field name="name">Insignia</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_mokka" model="fleet.vehicle.model">
          <field name="name">Mokka</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_antara" model="fleet.vehicle.model">
          <field name="name">Antara</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_ampera" model="fleet.vehicle.model">
          <field name="name">Ampera</field>
          <field name="brand_id" ref="brand_opel" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_a1" model="fleet.vehicle.model">
          <field name="name">A1</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_a3" model="fleet.vehicle.model">
          <field name="name">A3</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_a4" model="fleet.vehicle.model">
          <field name="name">A4</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_a5" model="fleet.vehicle.model">
          <field name="name">A5</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_a6" model="fleet.vehicle.model">
          <field name="name">A6</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_a7" model="fleet.vehicle.model">
          <field name="name">A7</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_a8" model="fleet.vehicle.model">
          <field name="name">A8</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_q3" model="fleet.vehicle.model">
          <field name="name">Q3</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_q5" model="fleet.vehicle.model">
          <field name="name">Q5</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_q7" model="fleet.vehicle.model">
          <field name="name">Q7</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_tt" model="fleet.vehicle.model">
          <field name="name">TT</field>
          <field name="brand_id" ref="brand_audi" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_serie1" model="fleet.vehicle.model">
          <field name="name">Serie 1</field>
          <field name="brand_id" ref="brand_bmw" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_serie3" model="fleet.vehicle.model">
          <field name="name">Serie 3</field>
          <field name="brand_id" ref="brand_bmw" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_serie5" model="fleet.vehicle.model">
          <field name="name">Serie 5</field>
          <field name="brand_id" ref="brand_bmw" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_serie6" model="fleet.vehicle.model">
          <field name="name">Serie 6</field>
          <field name="brand_id" ref="brand_bmw" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_serie7" model="fleet.vehicle.model">
          <field name="name">Serie 7</field>
          <field name="brand_id" ref="brand_bmw" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_seriex" model="fleet.vehicle.model">
          <field name="name">Serie X</field>
          <field name="brand_id" ref="brand_bmw" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_seriez4" model="fleet.vehicle.model">
          <field name="name">Serie Z4</field>
          <field name="brand_id" ref="brand_bmw" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_seriem" model="fleet.vehicle.model">
          <field name="name">Serie M</field>
          <field name="brand_id" ref="brand_bmw" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_seriehybrid" model="fleet.vehicle.model">
          <field name="name">Serie Hybrid</field>
          <field name="brand_id" ref="brand_bmw" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classa" model="fleet.vehicle.model">
          <field name="name">Class A</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classb" model="fleet.vehicle.model">
          <field name="name">Class B</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classc" model="fleet.vehicle.model">
          <field name="name">Class C</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classcl" model="fleet.vehicle.model">
          <field name="name">Class CL</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classcls" model="fleet.vehicle.model">
          <field name="name">Class CLS</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classe" model="fleet.vehicle.model">
          <field name="name">Class E</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classm" model="fleet.vehicle.model">
          <field name="name">Class M</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classgl" model="fleet.vehicle.model">
          <field name="name">Class GL</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classglk" model="fleet.vehicle.model">
          <field name="name">Class GLK</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classr" model="fleet.vehicle.model">
          <field name="name">Class R</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classs" model="fleet.vehicle.model">
          <field name="name">Class S</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classslk" model="fleet.vehicle.model">
          <field name="name">Class SLK</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
      <record id="model_classsls" model="fleet.vehicle.model">
          <field name="name">SLS</field>
          <field name="brand_id" ref="brand_mercedes" />
          <field name="vehicle_type">car</field>
      </record>
</odoo>

```

## File: data\fleet_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record forcecreate="True" id="ir_cron_contract_costs_generator" model="ir.cron">
            <field name="name">Fleet: Generate contracts costs based on costs frequency</field>
            <field name="model_id" ref="model_fleet_vehicle_log_contract"/>
            <field name="state">code</field>
            <field name="code">model.run_scheduler()</field>
            <field name="user_id" ref="base.user_root" />
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
            <field eval="False" name="doall" />
        </record>

        <record id="fleet_vehicle_state_new_request" model="fleet.vehicle.state">
            <field name="name">New Request</field>
            <field name="sequence">4</field>
        </record>

        <record id="fleet_vehicle_state_to_order" model="fleet.vehicle.state">
            <field name="name">To Order</field>
            <field name="sequence">5</field>
        </record>

        <record id="fleet_vehicle_state_registered" model="fleet.vehicle.state">
            <field name="name">Registered</field>
            <field name="sequence">7</field>
        </record>

        <record id="fleet_vehicle_state_downgraded" model="fleet.vehicle.state">
            <field name="name">Downgraded</field>
            <field name="sequence">8</field>
        </record>
    </data>
</odoo>

```

## File: data\fleet_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
      <!--Users-->
      <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[(4, ref('fleet.fleet_group_manager'))]" />
      </record>

        <record id="fleet_vehicle_state_ordered" model="fleet.vehicle.state">
            <field name="name">Ordered</field>
            <field name="sequence">6</field>
        </record>

      <record id="fleet_vehicle_state_reserve" model="fleet.vehicle.state">
          <field name="name">Reserve</field>
          <field name="sequence">9</field>
      </record>

      <record id="fleet_vehicle_state_waiting_list" model="fleet.vehicle.state">
          <field name="name">Waiting List</field>
          <field name="sequence">10</field>
      </record>

      <record id="type_service_service_1" model="fleet.service.type">
          <field name="name">Calculation Benefit In Kind</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_2" model="fleet.service.type">
          <field name="name">Depreciation and Interests</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_3" model="fleet.service.type">
          <field name="name">Tax roll</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_5" model="fleet.service.type">
          <field name="name">Summer tires</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_6" model="fleet.service.type">
          <field name="name">Snow tires</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_7" model="fleet.service.type">
          <field name="name">Repair and maintenance</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_8" model="fleet.service.type">
          <field name="name">Assistance</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_9" model="fleet.service.type">
          <field name="name">Replacement Vehicle</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_10" model="fleet.service.type">
          <field name="name">Management Fee</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_11" model="fleet.service.type">
          <field name="name">Rent (Excluding VAT)</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_12" model="fleet.service.type">
          <field name="name">Entry into service tax</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_13" model="fleet.service.type">
          <field name="name">Total expenses (Excluding VAT)</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_14" model="fleet.service.type">
          <field name="name">Residual value (Excluding VAT)</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_15" model="fleet.service.type">
          <field name="name">Options</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_16" model="fleet.service.type">
          <field name="name">Emissions</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_17" model="fleet.service.type">
          <field name="name">Touring Assistance</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_service_18" model="fleet.service.type">
          <field name="name">Residual value in %</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_1" model="fleet.service.type">
          <field name="name">A/C Compressor Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_2" model="fleet.service.type">
          <field name="name">A/C Condenser Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_3" model="fleet.service.type">
          <field name="name">A/C Diagnosis</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_4" model="fleet.service.type">
          <field name="name">A/C Evaporator Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_5" model="fleet.service.type">
          <field name="name">A/C Recharge</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_6" model="fleet.service.type">
          <field name="name">Air Filter Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_7" model="fleet.service.type">
          <field name="name">Alternator Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_7" model="fleet.service.type">
          <field name="name">Ball Joint Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_9" model="fleet.service.type">
          <field name="name">Battery Inspection</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_10" model="fleet.service.type">
          <field name="name">Battery Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_11" model="fleet.service.type">
          <field name="name">Brake Caliper Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_12" model="fleet.service.type">
          <field name="name">Brake Inspection</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_13" model="fleet.service.type">
          <field name="name">Brake Pad(s) Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_14" model="fleet.service.type">
          <field name="name">Car Wash</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_15" model="fleet.service.type">
          <field name="name">Catalytic Converter Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_16" model="fleet.service.type">
          <field name="name">Charging System Diagnosis</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_17" model="fleet.service.type">
          <field name="name">Door Window Motor/Regulator Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_18" model="fleet.service.type">
          <field name="name">Engine Belt Inspection</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_19" model="fleet.service.type">
          <field name="name">Engine Coolant Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_20" model="fleet.service.type">
          <field name="name">Engine/Drive Belt(s) Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_21" model="fleet.service.type">
          <field name="name">Exhaust Manifold Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_22" model="fleet.service.type">
          <field name="name">Fuel Injector Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_23" model="fleet.service.type">
          <field name="name">Fuel Pump Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_24" model="fleet.service.type">
          <field name="name">Head Gasket(s) Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_25" model="fleet.service.type">
          <field name="name">Heater Blower Motor Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_26" model="fleet.service.type">
          <field name="name">Heater Control Valve Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_27" model="fleet.service.type">
          <field name="name">Heater Core Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_28" model="fleet.service.type">
          <field name="name">Heater Hose Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_29" model="fleet.service.type">
          <field name="name">Ignition Coil Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_30" model="fleet.service.type">
          <field name="name">Intake Manifold Gasket Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_31" model="fleet.service.type">
          <field name="name">Oil Change</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_32" model="fleet.service.type">
          <field name="name">Oil Pump Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_33" model="fleet.service.type">
          <field name="name">Other Maintenance</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_34" model="fleet.service.type">
          <field name="name">Oxygen Sensor Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_35" model="fleet.service.type">
          <field name="name">Power Steering Hose Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_36" model="fleet.service.type">
          <field name="name">Power Steering Pump Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_37" model="fleet.service.type">
          <field name="name">Radiator Repair</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_38" model="fleet.service.type">
          <field name="name">Resurface Rotors</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_39" model="fleet.service.type">
          <field name="name">Rotate Tires</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_40" model="fleet.service.type">
          <field name="name">Rotor Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_41" model="fleet.service.type">
          <field name="name">Spark Plug Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_42" model="fleet.service.type">
          <field name="name">Starter Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_43" model="fleet.service.type">
          <field name="name">Thermostat Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_44" model="fleet.service.type">
          <field name="name">Tie Rod End Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_45" model="fleet.service.type">
          <field name="name">Tire Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_46" model="fleet.service.type">
          <field name="name">Tire Service</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_47" model="fleet.service.type">
          <field name="name">Transmission Filter Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_48" model="fleet.service.type">
          <field name="name">Transmission Fluid Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_49" model="fleet.service.type">
          <field name="name">Transmission Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_50" model="fleet.service.type">
          <field name="name">Water Pump Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_51" model="fleet.service.type">
          <field name="name">Wheel Alignment</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_52" model="fleet.service.type">
          <field name="name">Wheel Bearing Replacement</field>
          <field name="category">service</field>
      </record>

      <record id="type_service_53" model="fleet.service.type">
          <field name="name">Windshield Wiper(s) Replacement</field>
          <field name="category">service</field>
      </record>


      <record id="type_contract_omnium" model="fleet.service.type">
          <field name="name">Omnium</field>
          <field name="category">contract</field>
      </record>

      <record id="type_contract_leasing" model="fleet.service.type">
          <field name="name">Leasing</field>
          <field name="category">contract</field>
      </record>

       <record id="type_contract_repairing" model="fleet.service.type">
          <field name="name">Repairing</field>
          <field name="category">contract</field>
      </record>

      <record id="type_service_refueling" model="fleet.service.type">
          <field name="name">Refueling</field>
          <field name="category">service</field>
      </record>

      <record id="vehicle_tag_junior" model="fleet.vehicle.tag" >
        <field name="name">Junior</field>
        <field name="color" eval="1"/>
      </record>

      <record id="vehicle_tag_senior" model="fleet.vehicle.tag" >
        <field name="name">Senior</field>
        <field name="color" eval="2"/>
      </record>

      <record id="vehicle_tag_leasing" model="fleet.vehicle.tag" >
        <field name="name">Employee Car</field>
        <field name="color" eval="3"/>
      </record>

      <record id="vehicle_tag_purchased" model="fleet.vehicle.tag" >
        <field name="name">Purchased</field>
        <field name="color" eval="4"/>
      </record>

      <record id="model_category_1" model="fleet.vehicle.model.category">
        <field name="name">Break</field>
      </record>

      <record id="model_category_2" model="fleet.vehicle.model.category">
        <field name="name">SUV</field>
      </record>

      <record id="model_category_3" model="fleet.vehicle.model.category">
        <field name="name">Sport Car</field>
      </record>

      <record id="model_category_4" model="fleet.vehicle.model.category">
        <field name="name">Compact</field>
      </record>

      <record id="vehicle_1" model="fleet.vehicle">
          <field name="license_plate">1-ACK-205</field>
          <field name="vin_sn">5454541</field>
          <field name="model_id" ref="model_astra"/>
          <field name="color">Black</field>
          <field name="location">Grand-Rosiere</field>
          <field name="doors">5</field>
          <field name="driver_id" ref="base.partner_demo" />
          <field name="acquisition_date" eval="(DateTime.now() - timedelta(days=336)).strftime('%Y-%m-%d')" />
          <field name="state_id" ref="fleet_vehicle_state_registered"/>
          <field name="odometer_unit">kilometers</field>
          <field name="car_value">20000</field>
          <field eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_purchased'),ref('fleet.vehicle_tag_senior')])]" name="tag_ids"/>
      </record>

      <record id="vehicle_2" model="fleet.vehicle">
          <field name="license_plate">1-SYN-404</field>
          <field name="vin_sn">1337</field>
          <field name="model_id" ref="model_corsa"/>
          <field name="color">Red</field>
          <field name="location">Grand-Rosiere</field>
          <field name="doors">5</field>
          <field name="driver_id" ref="base.res_partner_address_25" />
          <field name="acquisition_date" eval="(DateTime.now() - timedelta(days=233)).strftime('%Y-%m-%d')" />
          <field name="state_id" ref="fleet_vehicle_state_downgraded"/>
          <field name="odometer_unit">kilometers</field>
          <field name="car_value">16000</field>
          <field eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_purchased'),ref('fleet.vehicle_tag_junior')])]" name="tag_ids"/>
      </record>

      <record id="vehicle_3" model="fleet.vehicle">
          <field name="license_plate">1-BMW-001</field>
          <field name="vin_sn">54818</field>
          <field name="model_id" ref="model_serie1"/>
          <field name="color">Titanium Grey</field>
          <field name="location">Grand-Rosiere</field>
          <field name="doors">3</field>
          <field name="driver_id" ref="base.res_partner_address_17" />
          <field name="acquisition_date" eval="time.strftime('%Y-%m-%d 2:00:00')" />
          <field name="state_id" ref="fleet_vehicle_state_registered"/>
          <field name="odometer_unit">kilometers</field>
          <field name="car_value">20000</field>
          <field eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_purchased'),ref('fleet.vehicle_tag_senior')])]" name="tag_ids"/>
      </record>

       <record id="vehicle_4" model="fleet.vehicle">
          <field name="license_plate">1-AUD-001</field>
          <field name="vin_sn">455257985</field>
          <field name="model_id" ref="model_a1"/>
          <field name="color">White</field>
          <field name="location">Grand-Rosiere</field>
          <field name="doors">3</field>
          <field name="driver_id" ref="base.res_partner_address_16" />
          <field name="acquisition_date" eval="time.strftime('%Y-%m-%d 2:00:00')" />
          <field name="state_id" ref="fleet_vehicle_state_registered"/>
          <field name="odometer_unit">kilometers</field>
          <field name="car_value">20000</field>
          <field eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_purchased'),ref('fleet.vehicle_tag_senior')])]" name="tag_ids"/>
      </record>

      <record id="vehicle_5" model="fleet.vehicle">
          <field name="license_plate">1-MER-001</field>
          <field name="vin_sn">789546128</field>
          <field name="model_id" ref="model_classa"/>
          <field name="color">Brown</field>
          <field name="location">Grand-Rosiere</field>
          <field name="doors">5</field>
          <field name="driver_id" ref="base.res_partner_address_15" />
          <field name="acquisition_date" eval="time.strftime('%Y-%m-%d 2:00:00')" />
          <field name="state_id" ref="fleet_vehicle_state_registered"/>
          <field name="odometer_unit">kilometers</field>
          <field name="car_value">18000</field>
          <field eval="[(6,0,[ref('vehicle_tag_leasing'),ref('fleet.vehicle_tag_purchased'),ref('fleet.vehicle_tag_senior')])]" name="tag_ids"/>
      </record>

      <record id="log_odometer_1" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=336)).strftime('%Y-%m-%d')" />
          <field name="value">0</field>
      </record>

      <record id="log_odometer_2" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=304)).strftime('%Y-%m-%d')" />
          <field name="value">658</field>
      </record>

      <record id="log_odometer_3" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=289)).strftime('%Y-%m-%d')" />
          <field name="value">1360</field>
      </record>

      <record id="log_odometer_4" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=260)).strftime('%Y-%m-%d')" />
          <field name="value">2044</field>
      </record>

      <record id="log_odometer_5" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=230)).strftime('%Y-%m-%d')" />
          <field name="value">2756</field>
      </record>

      <record id="log_odometer_6" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=185)).strftime('%Y-%m-%d')" />
          <field name="value">3410</field>
      </record>

      <record id="log_odometer_7" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=165)).strftime('%Y-%m-%d')" />
          <field name="value">3750</field>
      </record>

      <record id="log_odometer_8" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=140)).strftime('%Y-%m-%d')" />
          <field name="value">4115</field>
      </record>

      <record id="log_odometer_9" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=120)).strftime('%Y-%m-%d')" />
          <field name="value">4750</field>
      </record>

      <record id="log_odometer_10" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=105)).strftime('%Y-%m-%d')" />
          <field name="value">5171</field>
      </record>

      <record id="log_odometer_11" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=85)).strftime('%Y-%m-%d')" />
          <field name="value">5873</field>
      </record>

      <record id="log_odometer_12" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=55)).strftime('%Y-%m-%d')" />
          <field name="value">6571</field>
      </record>

      <record id="log_odometer_13" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=32)).strftime('%Y-%m-%d')" />
          <field name="value">7954</field>
      </record>

      <record id="log_odometer_14" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_1" />
          <field name="date" eval="(DateTime.now() - timedelta(days=2)).strftime('%Y-%m-%d')" />
          <field name="value">7981</field>
      </record>

      <record id="log_odometer_15" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=233)).strftime('%Y-%m-%d')" />
          <field name="value">0</field>
      </record>

      <record id="log_odometer_16" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=233)).strftime('%Y-%m-%d')" />
          <field name="value">702</field>
      </record>

      <record id="log_odometer_17" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=215)).strftime('%Y-%m-%d')" />
          <field name="value">1205.4</field>
      </record>

      <record id="log_odometer_18" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=200)).strftime('%Y-%m-%d')" />
          <field name="value">2122</field>
      </record>

      <record id="log_odometer_19" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=180)).strftime('%Y-%m-%d')" />
          <field name="value">2430</field>
      </record>

      <record id="log_odometer_20" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=165)).strftime('%Y-%m-%d')" />
          <field name="value">3015</field>
      </record>

      <record id="log_odometer_21" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=150)).strftime('%Y-%m-%d')" />
          <field name="value">3602.1</field>
      </record>

      <record id="log_odometer_22" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=124)).strftime('%Y-%m-%d')" />
          <field name="value">4205.5</field>
      </record>

      <record id="log_odometer_23" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=96)).strftime('%Y-%m-%d')" />
          <field name="value">4935</field>
      </record>

      <record id="log_odometer_24" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=80)).strftime('%Y-%m-%d')" />
          <field name="value">5555</field>
      </record>

      <record id="log_odometer_25" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=55)).strftime('%Y-%m-%d')" />
          <field name="value">5987</field>
      </record>

      <record id="log_odometer_26" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=45)).strftime('%Y-%m-%d')" />
          <field name="value">6571</field>
      </record>

      <record id="log_odometer_27" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=30)).strftime('%Y-%m-%d')" />
          <field name="value">7201.5</field>
      </record>

      <record id="log_odometer_28" model="fleet.vehicle.odometer">
          <field name="vehicle_id" ref="vehicle_2" />
          <field name="date" eval="(DateTime.now() - timedelta(days=10)).strftime('%Y-%m-%d')" />
          <field name="value">8001.2</field>
      </record>


      <record id="log_service_1" model="fleet.vehicle.log.services" >
        <field name="vehicle_id" ref="vehicle_2" />
        <field name="amount">650</field>
        <field name="service_type_id" ref="type_service_service_7"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=60)).strftime('%Y-%m-%d')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="inv_ref">4586</field>
        <field name="vendor_id" ref="base.res_partner_2" />
        <field name="notes">Usual vehicle repairing</field>
        <field name="state">done</field>
      </record>

      <record id="log_service_2" model="fleet.vehicle.log.services" >
        <field name="vehicle_id" ref="vehicle_2" />
        <field name="amount">350</field>
        <field name="service_type_id" ref="type_service_service_7"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=30)).strftime('%Y-%m-%d')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="inv_ref">4814</field>
        <field name="vendor_id" ref="base.res_partner_2" />
        <field name="notes">After crash repairing</field>
        <field name="state">done</field>
      </record>

      <record id="log_service_3" model="fleet.vehicle.log.services" >
        <field name="vehicle_id" ref="vehicle_1" />
        <field name="amount">513</field>
        <field name="service_type_id" ref="type_service_service_7"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=15)).strftime('%Y-%m-%d')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="inv_ref">124</field>
        <field name="vendor_id" ref="base.res_partner_2" />
        <field name="notes">Maintenance</field>
        <field name="state">done</field>
      </record>

      <record id="log_service_4" model="fleet.vehicle.log.services" >
        <field name="vehicle_id" ref="vehicle_3" />
        <field name="amount">412</field>
        <field name="service_type_id" ref="type_service_service_7"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=120)).strftime('%Y-%m-%d')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="inv_ref">20984</field>
        <field name="vendor_id" ref="base.res_partner_2" />
        <field name="notes">Maintenance</field>
        <field name="state">done</field>
      </record>

      <record id="log_service_5" model="fleet.vehicle.log.services" >
        <field name="vehicle_id" ref="vehicle_4" />
        <field name="amount">275</field>
        <field name="service_type_id" ref="type_service_service_7"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=100)).strftime('%Y-%m-%d')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="inv_ref">241</field>
        <field name="vendor_id" ref="base.res_partner_2" />
        <field name="notes">Maintenance</field>
        <field name="state">done</field>
      </record>

      <record id="log_service_6" model="fleet.vehicle.log.services" >
        <field name="vehicle_id" ref="vehicle_5" />
        <field name="amount">302</field>
        <field name="service_type_id" ref="type_service_service_7"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=65)).strftime('%Y-%m-%d')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="inv_ref">22513</field>
        <field name="vendor_id" ref="base.res_partner_2" />
        <field name="notes">Maintenance</field>
        <field name="state">done</field>
      </record>

      <record id="log_contract_1" model="fleet.vehicle.log.contract" >
        <field name="vehicle_id" ref="vehicle_2" />
        <field name="cost_subtype_id" ref="type_contract_leasing" />
        <field name="amount">0</field>
        <field name="name">Daily leasing contract</field>
        <field name="cost_generated">20</field>
        <field name="cost_frequency">daily</field>
        <field name="expiration_date" eval="(DateTime.now() - timedelta(days=233)).strftime('%Y-%m-%d')" />
        <field name="expiration_date" eval="(DateTime.now() + timedelta(5)).strftime('%Y-%m-%d')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="insurer_id" ref="base.res_partner_2" />
        <field name="notes">Daily leasing contract</field>
        <field name="state">open</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="[(6,0,[ref('fleet.type_contract_omnium'),ref('fleet.type_service_service_3'),ref('fleet.type_service_service_2')])]" name="service_ids"/>
      </record>

      <record id="log_contract_2" model="fleet.vehicle.log.contract" >
        <field name="vehicle_id" ref="vehicle_1" />
        <field name="cost_subtype_id" ref="type_contract_leasing" />
        <field name="amount">0</field>
        <field name="name">Weekly leasing contract</field>
        <field name="cost_generated">150</field>
        <field name="cost_frequency">weekly</field>
        <field name="date" eval="time.strftime('%Y-01-01')" />
        <field name="start_date" eval="(DateTime.now() - timedelta(days=289)).strftime('%Y-%m-%d')" />
        <field name="expiration_date" eval="(DateTime.now() + timedelta(-1)).strftime('%Y-%m-%d')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="insurer_id" ref="base.res_partner_2" />
        <field name="notes">Weekly leasing contract</field>
        <field name="state">open</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="[(6,0,[ref('fleet.type_contract_omnium'),ref('fleet.type_service_service_3'),ref('fleet.type_service_service_2')])]" name="service_ids"/>
      </record>

      <record id="log_contract_3" model="fleet.vehicle.log.contract" >
        <field name="vehicle_id" ref="vehicle_3" />
        <field name="cost_subtype_id" ref="type_contract_leasing" />
        <field name="amount">0</field>
        <field name="name">Monthly leasing</field>
        <field name="cost_generated">400</field>
        <field name="cost_frequency">monthly</field>
        <field name="date" eval="time.strftime('%Y-01-01')"/>
        <field name="start_date" eval="time.strftime('%Y-01-01')" />
        <field name="expiration_date" eval="time.strftime('%Y-12-31')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="insurer_id" ref="base.res_partner_2" />
        <field name="notes">Monthly leasing contract</field>
        <field name="state">open</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="[(6,0,[ref('fleet.type_contract_omnium'),ref('fleet.type_service_service_3'),ref('fleet.type_service_service_2')])]" name="service_ids"/>
      </record>

      <record id="log_contract_4" model="fleet.vehicle.log.contract" >
        <field name="vehicle_id" ref="vehicle_4" />
        <field name="cost_subtype_id" ref="type_contract_leasing" />
        <field name="amount">0</field>
        <field name="name">Yearly leasing</field>
        <field name="cost_generated">4000</field>
        <field name="cost_frequency">yearly</field>
        <field name="date" eval="time.strftime('%Y-01-01')" />
        <field name="start_date" eval="time.strftime('%Y-01-01')" />
        <field name="expiration_date" eval="time.strftime('%Y-12-31')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="insurer_id" ref="base.res_partner_2" />
        <field name="notes">Yearly leasing contract</field>
        <field name="state">open</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="[(6,0,[ref('fleet.type_contract_omnium'),ref('fleet.type_service_service_3'),ref('fleet.type_service_service_2')])]" name="service_ids"/>
      </record>

      <record id="log_contract_5" model="fleet.vehicle.log.contract" >
        <field name="vehicle_id" ref="vehicle_5" />
        <field name="cost_subtype_id" ref="type_contract_leasing" />
        <field name="amount">17000</field>
        <field name="name">Unique leasing</field>
        <field name="cost_generated">0</field>
        <field name="cost_frequency">no</field>
        <field name="date" eval="(DateTime.now() - timedelta(days=300)).strftime('%Y-%m-%d')" />
        <field name="start_date" eval="(DateTime.now() - timedelta(days=300)).strftime('%Y-%m-%d')" />
        <field name="expiration_date" eval="(DateTime.now() + timedelta(-60)).strftime('%Y-%m-%d')" />
        <field name="purchaser_id" ref="base.res_partner_address_18" />
        <field name="insurer_id" ref="base.res_partner_2" />
        <field name="notes">Unique leasing contract</field>
        <field name="state">open</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="[(6,0,[ref('fleet.type_contract_omnium'),ref('fleet.type_service_service_3'),ref('fleet.type_service_service_2')])]" name="service_ids"/>
      </record>
</odoo>

```

## File: data\mail_activity_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_act_fleet_contract_to_renew" model="mail.activity.type">
            <field name="name">Contract to Renew</field>
            <field name="icon">fa-car</field>
            <field name="res_model">fleet.vehicle.log.contract</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mt_fleet_driver_updated" model="mail.message.subtype">
            <field name="name">Changed Driver</field>
            <field name="sequence">0</field>
            <field name="res_model">fleet.vehicle</field>
            <field name="default" eval="True"/>
            <field name="description">Changed Driver</field>
        </record>
    </data>
</odoo>

```

## File: models\fleet_service_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class FleetServiceType(models.Model):
    _name = 'fleet.service.type'
    _description = 'Fleet Service Type'
    _order = 'name'

    name = fields.Char(required=True, translate=True)
    category = fields.Selection([
        ('contract', 'Contract'),
        ('service', 'Service')
        ], 'Category', required=True, help='Choose whether the service refer to contracts, vehicle services or both')

```

## File: models\fleet_vehicle.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.osv import expression
from odoo.addons.fleet.models.fleet_vehicle_model import FUEL_TYPES


#Some fields don't have the exact same name
MODEL_FIELDS_TO_VEHICLE = {
    'transmission': 'transmission', 'model_year': 'model_year', 'electric_assistance': 'electric_assistance',
    'color': 'color', 'seats': 'seats', 'doors': 'doors', 'trailer_hook': 'trailer_hook',
    'default_co2': 'co2', 'co2_standard': 'co2_standard', 'default_fuel_type': 'fuel_type',
    'power': 'power', 'horsepower': 'horsepower', 'horsepower_tax': 'horsepower_tax', 'category_id': 'category_id',
}

class FleetVehicle(models.Model):
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _name = 'fleet.vehicle'
    _description = 'Vehicle'
    _order = 'license_plate asc, acquisition_date asc'
    _rec_names_search = ['name', 'driver_id.name']

    def _get_default_state(self):
        state = self.env.ref('fleet.fleet_vehicle_state_registered', raise_if_not_found=False)
        return state if state and state.id else False

    name = fields.Char(compute="_compute_vehicle_name", store=True)
    description = fields.Html("Vehicle Description")
    active = fields.Boolean('Active', default=True, tracking=True)
    manager_id = fields.Many2one(
        'res.users', 'Fleet Manager',
        domain=lambda self: [('groups_id', 'in', self.env.ref('fleet.fleet_group_manager').id)],
    )
    company_id = fields.Many2one(
        'res.company', 'Company',
        default=lambda self: self.env.company,
    )
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')
    country_id = fields.Many2one('res.country', related='company_id.country_id')
    country_code = fields.Char(related='country_id.code', depends=['country_id'])
    license_plate = fields.Char(tracking=True,
        help='License plate number of the vehicle (i = plate number for a car)')
    vin_sn = fields.Char('Chassis Number', help='Unique number written on the vehicle motor (VIN/SN number)', copy=False)
    trailer_hook = fields.Boolean(default=False, string='Trailer Hitch', compute='_compute_model_fields', store=True, readonly=False)
    driver_id = fields.Many2one('res.partner', 'Driver', tracking=True, help='Driver address of the vehicle', copy=False)
    future_driver_id = fields.Many2one('res.partner', 'Future Driver', tracking=True, help='Next Driver Address of the vehicle', copy=False, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    model_id = fields.Many2one('fleet.vehicle.model', 'Model',
        tracking=True, required=True)

    brand_id = fields.Many2one('fleet.vehicle.model.brand', 'Brand', related="model_id.brand_id", store=True, readonly=False)
    log_drivers = fields.One2many('fleet.vehicle.assignation.log', 'vehicle_id', string='Assignment Logs')
    log_services = fields.One2many('fleet.vehicle.log.services', 'vehicle_id', 'Services Logs')
    log_contracts = fields.One2many('fleet.vehicle.log.contract', 'vehicle_id', 'Contracts')
    contract_count = fields.Integer(compute="_compute_count_all", string='Contract Count')
    service_count = fields.Integer(compute="_compute_count_all", string='Services')
    odometer_count = fields.Integer(compute="_compute_count_all", string='Odometer')
    history_count = fields.Integer(compute="_compute_count_all", string="Drivers History Count")
    next_assignation_date = fields.Date('Assignment Date', help='This is the date at which the car will be available, if not set it means available instantly')
    acquisition_date = fields.Date('Registration Date', required=False,
        default=fields.Date.today, help='Date of vehicle registration')
    write_off_date = fields.Date('Cancellation Date', tracking=True, help="Date when the vehicle's license plate has been cancelled/removed.")
    first_contract_date = fields.Date(string="First Contract Date", default=fields.Date.today)
    color = fields.Char(help='Color of the vehicle', compute='_compute_model_fields', store=True, readonly=False)
    state_id = fields.Many2one('fleet.vehicle.state', 'State',
        default=_get_default_state, group_expand='_read_group_stage_ids',
        tracking=True,
        help='Current state of the vehicle', ondelete="set null")
    location = fields.Char(help='Location of the vehicle (garage, ...)')
    seats = fields.Integer('Seats Number', help='Number of seats of the vehicle', compute='_compute_model_fields', store=True, readonly=False)
    model_year = fields.Char('Model Year', help='Year of the model', compute='_compute_model_fields', store=True, readonly=False)
    doors = fields.Integer('Doors Number', help='Number of doors of the vehicle', compute='_compute_model_fields', store=True, readonly=False)
    tag_ids = fields.Many2many('fleet.vehicle.tag', 'fleet_vehicle_vehicle_tag_rel', 'vehicle_tag_id', 'tag_id', 'Tags', copy=False)
    odometer = fields.Float(compute='_get_odometer', inverse='_set_odometer', string='Last Odometer',
        help='Odometer measure of the vehicle at the moment of this log')
    odometer_unit = fields.Selection([
        ('kilometers', 'km'),
        ('miles', 'mi')
        ], 'Odometer Unit', default='kilometers', required=True)
    transmission = fields.Selection(
        [('manual', 'Manual'), ('automatic', 'Automatic')], 'Transmission',
        compute='_compute_model_fields', store=True, readonly=False)
    fuel_type = fields.Selection(FUEL_TYPES, 'Fuel Type', compute='_compute_model_fields', store=True, readonly=False)
    horsepower = fields.Integer(compute='_compute_model_fields', store=True, readonly=False)
    horsepower_tax = fields.Float('Horsepower Taxation', compute='_compute_model_fields', store=True, readonly=False)
    power = fields.Integer('Power', help='Power in kW of the vehicle', compute='_compute_model_fields', store=True, readonly=False)
    co2 = fields.Float('CO2 Emissions', help='CO2 emissions of the vehicle', compute='_compute_model_fields', store=True, readonly=False, tracking=True)
    co2_standard = fields.Char('CO2 Standard', compute='_compute_model_fields', store=True, readonly=False)
    category_id = fields.Many2one('fleet.vehicle.model.category', 'Category', compute='_compute_model_fields', store=True, readonly=False)
    image_128 = fields.Image(related='model_id.image_128', readonly=True)
    contract_renewal_due_soon = fields.Boolean(compute='_compute_contract_reminder', search='_search_contract_renewal_due_soon',
        string='Has Contracts to renew')
    contract_renewal_overdue = fields.Boolean(compute='_compute_contract_reminder', search='_search_get_overdue_contract_reminder',
        string='Has Contracts Overdue')
    contract_renewal_name = fields.Text(compute='_compute_contract_reminder', string='Name of contract to renew soon')
    contract_renewal_total = fields.Text(compute='_compute_contract_reminder', string='Total of contracts due or overdue minus one')
    contract_state = fields.Selection(
        [('futur', 'Incoming'),
         ('open', 'In Progress'),
         ('expired', 'Expired'),
         ('closed', 'Closed')
        ], string='Last Contract State', compute='_compute_contract_reminder', required=False)
    car_value = fields.Float(string="Catalog Value (VAT Incl.)")
    net_car_value = fields.Float(string="Purchase Value")
    residual_value = fields.Float()
    plan_to_change_car = fields.Boolean(related='driver_id.plan_to_change_car', store=True, readonly=False)
    plan_to_change_bike = fields.Boolean(related='driver_id.plan_to_change_bike', store=True, readonly=False)
    vehicle_type = fields.Selection(related='model_id.vehicle_type')
    frame_type = fields.Selection([('diamant', 'Diamant'), ('trapez', 'Trapez'), ('wave', 'Wave')], string="Bike Frame Type")
    electric_assistance = fields.Boolean(compute='_compute_model_fields', store=True, readonly=False)
    frame_size = fields.Float()
    service_activity = fields.Selection([
        ('none', 'None'),
        ('overdue', 'Overdue'),
        ('today', 'Today'),
    ], compute='_compute_service_activity')

    @api.depends('log_services')
    def _compute_service_activity(self):
        for vehicle in self:
            activities_state = set(state for state in vehicle.log_services.mapped('activity_state') if state and state != 'planned')
            vehicle.service_activity = sorted(activities_state)[0] if activities_state else 'none'

    @api.depends('model_id')
    def _compute_model_fields(self):
        '''
        Copies all the related fields from the model to the vehicle
        '''
        model_values = dict()
        for vehicle in self.filtered('model_id'):
            if vehicle.model_id.id in model_values:
                write_vals = model_values[vehicle.model_id.id]
            else:
                # copy if value is truthy
                write_vals = {MODEL_FIELDS_TO_VEHICLE[key]: vehicle.model_id[key] for key in MODEL_FIELDS_TO_VEHICLE\
                    if vehicle.model_id[key]}
                model_values[vehicle.model_id.id] = write_vals
            vehicle.update(write_vals)

    @api.depends('model_id.brand_id.name', 'model_id.name', 'license_plate')
    def _compute_vehicle_name(self):
        for record in self:
            record.name = (record.model_id.brand_id.name or '') + '/' + (record.model_id.name or '') + '/' + (record.license_plate or _('No Plate'))

    def _get_odometer(self):
        FleetVehicalOdometer = self.env['fleet.vehicle.odometer']
        for record in self:
            vehicle_odometer = FleetVehicalOdometer.search([('vehicle_id', '=', record.id)], limit=1, order='value desc')
            if vehicle_odometer:
                record.odometer = vehicle_odometer.value
            else:
                record.odometer = 0

    def _set_odometer(self):
        for record in self:
            if record.odometer:
                date = fields.Date.context_today(record)
                data = {'value': record.odometer, 'date': date, 'vehicle_id': record.id}
                self.env['fleet.vehicle.odometer'].create(data)

    def _compute_count_all(self):
        Odometer = self.env['fleet.vehicle.odometer']
        LogService = self.env['fleet.vehicle.log.services'].with_context(active_test=False)
        LogContract = self.env['fleet.vehicle.log.contract'].with_context(active_test=False)
        History = self.env['fleet.vehicle.assignation.log']
        odometers_data = Odometer.read_group([('vehicle_id', 'in', self.ids)], ['vehicle_id'], ['vehicle_id'])
        services_data = LogService.read_group([('vehicle_id', 'in', self.ids)], ['vehicle_id', 'active'], ['vehicle_id', 'active'], lazy=False)
        logs_data = LogContract.read_group([('vehicle_id', 'in', self.ids), ('state', '!=', 'closed')], ['vehicle_id', 'active'], ['vehicle_id', 'active'], lazy=False)
        histories_data = History.read_group([('vehicle_id', 'in', self.ids)], ['vehicle_id'], ['vehicle_id'])

        mapped_odometer_data = defaultdict(lambda: 0)
        mapped_service_data = defaultdict(lambda: defaultdict(lambda: 0))
        mapped_log_data = defaultdict(lambda: defaultdict(lambda: 0))
        mapped_history_data = defaultdict(lambda: 0)

        for odometer_data in odometers_data:
            mapped_odometer_data[odometer_data['vehicle_id'][0]] = odometer_data['vehicle_id_count']
        for service_data in services_data:
            mapped_service_data[service_data['vehicle_id'][0]][service_data['active']] = service_data['__count']
        for log_data in logs_data:
            mapped_log_data[log_data['vehicle_id'][0]][log_data['active']] = log_data['__count']
        for history_data in histories_data:
            mapped_history_data[history_data['vehicle_id'][0]] = history_data['vehicle_id_count']

        for vehicle in self:
            vehicle.odometer_count = mapped_odometer_data[vehicle.id]
            vehicle.service_count = mapped_service_data[vehicle.id][vehicle.active]
            vehicle.contract_count = mapped_log_data[vehicle.id][vehicle.active]
            vehicle.history_count = mapped_history_data[vehicle.id]

    @api.depends('log_contracts')
    def _compute_contract_reminder(self):
        params = self.env['ir.config_parameter'].sudo()
        delay_alert_contract = int(params.get_param('hr_fleet.delay_alert_contract', default=30))
        for record in self:
            overdue = False
            due_soon = False
            total = 0
            name = ''
            state = ''
            for element in record.log_contracts:
                if element.state in ('open', 'expired') and element.expiration_date:
                    current_date_str = fields.Date.context_today(record)
                    due_time_str = element.expiration_date
                    current_date = fields.Date.from_string(current_date_str)
                    due_time = fields.Date.from_string(due_time_str)
                    diff_time = (due_time - current_date).days
                    if diff_time < 0:
                        overdue = True
                        total += 1
                    if diff_time < delay_alert_contract:
                        due_soon = True
                        total += 1
                    if overdue or due_soon:
                        log_contract = self.env['fleet.vehicle.log.contract'].search([
                            ('vehicle_id', '=', record.id),
                            ('state', 'in', ('open', 'expired'))
                            ], limit=1, order='expiration_date asc')
                        if log_contract:
                            # we display only the name of the oldest overdue/due soon contract
                            name = log_contract.name
                            state = log_contract.state

            record.contract_renewal_overdue = overdue
            record.contract_renewal_due_soon = due_soon
            record.contract_renewal_total = total - 1  # we remove 1 from the real total for display purposes
            record.contract_renewal_name = name
            record.contract_state = state

    def _get_analytic_name(self):
        # This function is used in fleet_account and is overrided in l10n_be_hr_payroll_fleet
        return self.license_plate or _('No plate')

    def _search_contract_renewal_due_soon(self, operator, value):
        params = self.env['ir.config_parameter'].sudo()
        delay_alert_contract = int(params.get_param('hr_fleet.delay_alert_contract', default=30))
        res = []
        assert operator in ('=', '!=', '<>') and value in (True, False), 'Operation not supported'
        if (operator == '=' and value is True) or (operator in ('<>', '!=') and value is False):
            search_operator = 'in'
        else:
            search_operator = 'not in'
        today = fields.Date.context_today(self)
        datetime_today = fields.Datetime.from_string(today)
        limit_date = fields.Datetime.to_string(datetime_today + relativedelta(days=+delay_alert_contract))
        res_ids = self.env['fleet.vehicle.log.contract'].search([
            ('expiration_date', '>', today),
            ('expiration_date', '<', limit_date),
            ('state', 'in', ['open', 'expired'])
        ]).mapped('vehicle_id').ids
        res.append(('id', search_operator, res_ids))
        return res

    def _search_get_overdue_contract_reminder(self, operator, value):
        res = []
        assert operator in ('=', '!=', '<>') and value in (True, False), 'Operation not supported'
        if (operator == '=' and value is True) or (operator in ('<>', '!=') and value is False):
            search_operator = 'in'
        else:
            search_operator = 'not in'
        today = fields.Date.context_today(self)
        res_ids = self.env['fleet.vehicle.log.contract'].search([
            ('expiration_date', '!=', False),
            ('expiration_date', '<', today),
            ('state', 'in', ['open', 'expired'])
        ]).mapped('vehicle_id').ids
        res.append(('id', search_operator, res_ids))
        return res

    def _clean_vals_internal_user(self, vals):
        # Fleet administrator may not have rights to write on partner
        # related fields when the driver_id is a res.user.
        # This trick is used to prevent access right error.
        su_vals = {}
        if self.env.su:
            return su_vals
        if 'plan_to_change_car' in vals:
            su_vals['plan_to_change_car'] = vals.pop('plan_to_change_car')
        if 'plan_to_change_bike' in vals:
            su_vals['plan_to_change_bike'] = vals.pop('plan_to_change_bike')
        return su_vals

    @api.model_create_multi
    def create(self, vals_list):
        ptc_values = [self._clean_vals_internal_user(vals) for vals in vals_list]
        vehicles = super().create(vals_list)
        for vehicle, vals, ptc_value in zip(vehicles, vals_list, ptc_values):
            if ptc_value:
                vehicle.sudo().write(ptc_value)
            if 'driver_id' in vals and vals['driver_id']:
                vehicle.create_driver_history(vals)
            if 'future_driver_id' in vals and vals['future_driver_id']:
                state_waiting_list = self.env.ref('fleet.fleet_vehicle_state_waiting_list', raise_if_not_found=False)
                states = vehicle.mapped('state_id').ids
                if not state_waiting_list or state_waiting_list.id not in states:
                    future_driver = self.env['res.partner'].browse(vals['future_driver_id'])
                    if self.vehicle_type == 'bike':
                        future_driver.sudo().write({'plan_to_change_bike': True})
                    if self.vehicle_type == 'car':
                        future_driver.sudo().write({'plan_to_change_car': True})
        return vehicles

    def write(self, vals):
        if 'driver_id' in vals and vals['driver_id']:
            driver_id = vals['driver_id']
            for vehicle in self.filtered(lambda v: v.driver_id.id != driver_id):
                vehicle.create_driver_history(vals)
                if vehicle.driver_id:
                    vehicle.activity_schedule(
                        'mail.mail_activity_data_todo',
                        user_id=vehicle.manager_id.id or self.env.user.id,
                        note=_('Specify the End date of %s') % vehicle.driver_id.name)

        if 'future_driver_id' in vals and vals['future_driver_id']:
            state_waiting_list = self.env.ref('fleet.fleet_vehicle_state_waiting_list', raise_if_not_found=False)
            states = self.mapped('state_id').ids if 'state_id' not in vals else [vals['state_id']]
            if not state_waiting_list or state_waiting_list.id not in states:
                future_driver = self.env['res.partner'].browse(vals['future_driver_id'])
                if self.vehicle_type == 'bike':
                    future_driver.sudo().write({'plan_to_change_bike': True})
                if self.vehicle_type == 'car':
                    future_driver.sudo().write({'plan_to_change_car': True})

        if 'active' in vals and not vals['active']:
            self.env['fleet.vehicle.log.contract'].search([('vehicle_id', 'in', self.ids)]).active = False
            self.env['fleet.vehicle.log.services'].search([('vehicle_id', 'in', self.ids)]).active = False

        su_vals = self._clean_vals_internal_user(vals)
        if su_vals:
            self.sudo().write(su_vals)
        res = super(FleetVehicle, self).write(vals)
        return res

    def _get_driver_history_data(self, vals):
        self.ensure_one()
        return {
            'vehicle_id': self.id,
            'driver_id': vals['driver_id'],
            'date_start': fields.Date.today(),
        }

    def create_driver_history(self, vals):
        for vehicle in self:
            self.env['fleet.vehicle.assignation.log'].create(
                vehicle._get_driver_history_data(vals),
            )

    def action_accept_driver_change(self):
        # Find all the vehicles of the same type for which the driver is the future_driver_id
        # remove their driver_id and close their history using current date
        vehicles = self.search([('driver_id', 'in', self.mapped('future_driver_id').ids), ('vehicle_type', '=', self.vehicle_type)])
        vehicles.write({'driver_id': False})

        for vehicle in self:
            if vehicle.vehicle_type == 'bike':
                vehicle.future_driver_id.sudo().write({'plan_to_change_bike': False})
            if vehicle.vehicle_type == 'car':
                vehicle.future_driver_id.sudo().write({'plan_to_change_car': False})
            vehicle.driver_id = vehicle.future_driver_id
            vehicle.future_driver_id = False

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        return self.env['fleet.vehicle.state'].search([], order=order)

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if 'co2' in fields:
            fields.remove('co2')
        return super(FleetVehicle, self).read_group(domain, fields, groupby, offset, limit, orderby, lazy)

    def return_action_to_open(self):
        """ This opens the xml view specified in xml_id for the current vehicle """
        self.ensure_one()
        xml_id = self.env.context.get('xml_id')
        if xml_id:

            res = self.env['ir.actions.act_window']._for_xml_id('fleet.%s' % xml_id)
            res.update(
                context=dict(self.env.context, default_vehicle_id=self.id, group_by=False),
                domain=[('vehicle_id', '=', self.id)]
            )
            return res
        return False

    def act_show_log_cost(self):
        """ This opens log view to view and add new log for this vehicle, groupby default to only show effective costs
            @return: the costs log view
        """
        self.ensure_one()
        copy_context = dict(self.env.context)
        copy_context.pop('group_by', None)
        res = self.env['ir.actions.act_window']._for_xml_id('fleet.fleet_vehicle_costs_action')
        res.update(
            context=dict(copy_context, default_vehicle_id=self.id, search_default_parent_false=True),
            domain=[('vehicle_id', '=', self.id)]
        )
        return res

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'driver_id' in init_values or 'future_driver_id' in init_values:
            return self.env.ref('fleet.mt_fleet_driver_updated')
        return super(FleetVehicle, self)._track_subtype(init_values)

    def open_assignation_logs(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': 'Assignment Logs',
            'view_mode': 'tree',
            'res_model': 'fleet.vehicle.assignation.log',
            'domain': [('vehicle_id', '=', self.id)],
            'context': {'default_driver_id': self.driver_id.id, 'default_vehicle_id': self.id}
        }

```

## File: models\fleet_vehicle_assignation_log.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class FleetVehicleAssignationLog(models.Model):
    _name = "fleet.vehicle.assignation.log"
    _description = "Drivers history on a vehicle"
    _order = "create_date desc, date_start desc"

    vehicle_id = fields.Many2one('fleet.vehicle', string="Vehicle", required=True)
    driver_id = fields.Many2one('res.partner', string="Driver", required=True)
    date_start = fields.Date(string="Start Date")
    date_end = fields.Date(string="End Date")

```

## File: models\fleet_vehicle_log_contract.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import api, fields, models


class FleetVehicleLogContract(models.Model):
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _name = 'fleet.vehicle.log.contract'
    _description = 'Vehicle Contract'
    _order = 'state desc,expiration_date'

    def compute_next_year_date(self, strdate):
        oneyear = relativedelta(years=1)
        start_date = fields.Date.from_string(strdate)
        return fields.Date.to_string(start_date + oneyear)

    vehicle_id = fields.Many2one('fleet.vehicle', 'Vehicle', required=True, check_company=True)
    cost_subtype_id = fields.Many2one('fleet.service.type', 'Type', help='Cost type purchased with this cost', domain=[('category', '=', 'contract')])
    amount = fields.Monetary('Cost', tracking=True)
    date = fields.Date(help='Date when the cost has been executed')
    company_id = fields.Many2one('res.company', 'Company', default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')
    name = fields.Char(string='Name', compute='_compute_contract_name', store=True, readonly=False)
    active = fields.Boolean(default=True)
    user_id = fields.Many2one('res.users', 'Responsible', default=lambda self: self.env.user, index=True)
    start_date = fields.Date(
        'Contract Start Date', default=fields.Date.context_today,
        help='Date when the coverage of the contract begins')
    expiration_date = fields.Date(
        'Contract Expiration Date', default=lambda self:
        self.compute_next_year_date(fields.Date.context_today(self)),
        help='Date when the coverage of the contract expirates (by default, one year after begin date)')
    days_left = fields.Integer(compute='_compute_days_left', string='Warning Date')
    expires_today = fields.Boolean(compute='_compute_days_left')
    insurer_id = fields.Many2one('res.partner', 'Vendor')
    purchaser_id = fields.Many2one(related='vehicle_id.driver_id', string='Driver')
    ins_ref = fields.Char('Reference', size=64, copy=False)
    state = fields.Selection(
        [('futur', 'Incoming'),
         ('open', 'In Progress'),
         ('expired', 'Expired'),
         ('closed', 'Closed')
        ], 'Status', default='open', readonly=True,
        help='Choose whether the contract is still valid or not',
        tracking=True,
        copy=False)
    notes = fields.Html('Terms and Conditions', copy=False)
    cost_generated = fields.Monetary('Recurring Cost', tracking=True)
    cost_frequency = fields.Selection([
        ('no', 'No'),
        ('daily', 'Daily'),
        ('weekly', 'Weekly'),
        ('monthly', 'Monthly'),
        ('yearly', 'Yearly')
        ], 'Recurring Cost Frequency', default='monthly', required=True)
    service_ids = fields.Many2many('fleet.service.type', string="Included Services")

    @api.depends('vehicle_id.name', 'cost_subtype_id')
    def _compute_contract_name(self):
        for record in self:
            name = record.vehicle_id.name
            if name and record.cost_subtype_id.name:
                name = record.cost_subtype_id.name + ' ' + name
            record.name = name

    @api.depends('expiration_date', 'state')
    def _compute_days_left(self):
        """return a dict with as value for each contract an integer
        if contract is in an open state and is overdue, return 0
        if contract is in a closed state, return -1
        otherwise return the number of days before the contract expires
        """
        today = fields.Date.from_string(fields.Date.today())
        for record in self:
            if record.expiration_date and record.state in ['open', 'expired']:
                renew_date = fields.Date.from_string(record.expiration_date)
                diff_time = (renew_date - today).days
                record.days_left = diff_time if diff_time > 0 else 0
                record.expires_today = diff_time == 0
            else:
                record.days_left = -1
                record.expires_today = False

    def write(self, vals):
        res = super(FleetVehicleLogContract, self).write(vals)
        if 'start_date' in vals or 'expiration_date' in vals:
            date_today = fields.Date.today()
            future_contracts, running_contracts, expired_contracts = self.env[self._name], self.env[self._name], self.env[self._name]
            for contract in self.filtered(lambda c: c.start_date and c.state != 'closed'):
                if date_today < contract.start_date:
                    future_contracts |= contract
                elif not contract.expiration_date or contract.start_date <= date_today <= contract.expiration_date:
                    running_contracts |= contract
                else:
                    expired_contracts |= contract
            future_contracts.action_draft()
            running_contracts.action_open()
            expired_contracts.action_expire()
        if vals.get('expiration_date') or vals.get('user_id'):
            self.activity_reschedule(['fleet.mail_act_fleet_contract_to_renew'], date_deadline=vals.get('expiration_date'), new_user_id=vals.get('user_id'))
        return res

    def action_close(self):
        self.write({'state': 'closed'})

    def action_draft(self):
        self.write({'state': 'futur'})

    def action_open(self):
        self.write({'state': 'open'})

    def action_expire(self):
        self.write({'state': 'expired'})

    @api.model
    def scheduler_manage_contract_expiration(self):
        # This method is called by a cron task
        # It manages the state of a contract, possibly by posting a message on the vehicle concerned and updating its status
        params = self.env['ir.config_parameter'].sudo()
        delay_alert_contract = int(params.get_param('hr_fleet.delay_alert_contract', default=30))
        date_today = fields.Date.from_string(fields.Date.today())
        outdated_days = fields.Date.to_string(date_today + relativedelta(days=+delay_alert_contract))
        reminder_activity_type = self.env.ref('fleet.mail_act_fleet_contract_to_renew', raise_if_not_found=False) or self.env['mail.activity.type']
        nearly_expired_contracts = self.search([
            ('state', '=', 'open'),
            ('expiration_date', '<', outdated_days),
            ('user_id', '!=', False)
        ]
        ).filtered(
            lambda nec: reminder_activity_type not in nec.activity_ids.activity_type_id
        )

        for contract in nearly_expired_contracts:
            contract.activity_schedule(
                'fleet.mail_act_fleet_contract_to_renew', contract.expiration_date,
                user_id=contract.user_id.id)

        expired_contracts = self.search([('state', 'not in', ['expired', 'closed']), ('expiration_date', '<',fields.Date.today() )])
        expired_contracts.write({'state': 'expired'})

        futur_contracts = self.search([('state', 'not in', ['futur', 'closed']), ('start_date', '>', fields.Date.today())])
        futur_contracts.write({'state': 'futur'})

        now_running_contracts = self.search([('state', '=', 'futur'), ('start_date', '<=', fields.Date.today())])
        now_running_contracts.write({'state': 'open'})

    def run_scheduler(self):
        self.scheduler_manage_contract_expiration()

```

## File: models\fleet_vehicle_log_services.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class FleetVehicleLogServices(models.Model):
    _name = 'fleet.vehicle.log.services'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _rec_name = 'service_type_id'
    _description = 'Services for vehicles'

    active = fields.Boolean(default=True)
    vehicle_id = fields.Many2one('fleet.vehicle', 'Vehicle', required=True)
    manager_id = fields.Many2one('res.users', 'Fleet Manager', related='vehicle_id.manager_id', store=True)
    amount = fields.Monetary('Cost')
    description = fields.Char('Description')
    odometer_id = fields.Many2one('fleet.vehicle.odometer', 'Odometer', help='Odometer measure of the vehicle at the moment of this log')
    odometer = fields.Float(
        compute="_get_odometer", inverse='_set_odometer', string='Odometer Value',
        help='Odometer measure of the vehicle at the moment of this log')
    odometer_unit = fields.Selection(related='vehicle_id.odometer_unit', string="Unit", readonly=True)
    date = fields.Date(help='Date when the cost has been executed', default=fields.Date.context_today)
    company_id = fields.Many2one('res.company', 'Company', default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')
    purchaser_id = fields.Many2one('res.partner', string="Driver", compute='_compute_purchaser_id', readonly=False, store=True)
    inv_ref = fields.Char('Vendor Reference')
    vendor_id = fields.Many2one('res.partner', 'Vendor')
    notes = fields.Text()
    service_type_id = fields.Many2one(
        'fleet.service.type', 'Service Type', required=True,
        default=lambda self: self.env.ref('fleet.type_service_service_7', raise_if_not_found=False),
    )
    state = fields.Selection([
        ('new', 'New'),
        ('running', 'Running'),
        ('done', 'Done'),
        ('cancelled', 'Cancelled'),
    ], default='new', string='Stage', group_expand='_expand_states')

    def _get_odometer(self):
        self.odometer = 0
        for record in self:
            if record.odometer_id:
                record.odometer = record.odometer_id.value

    def _set_odometer(self):
        for record in self:
            if not record.odometer:
                raise UserError(_('Emptying the odometer value of a vehicle is not allowed.'))
            odometer = self.env['fleet.vehicle.odometer'].create({
                'value': record.odometer,
                'date': record.date or fields.Date.context_today(record),
                'vehicle_id': record.vehicle_id.id
            })
            self.odometer_id = odometer

    @api.model_create_multi
    def create(self, vals_list):
        for data in vals_list:
            if 'odometer' in data and not data['odometer']:
                # if received value for odometer is 0, then remove it from the
                # data as it would result to the creation of a
                # odometer log with 0, which is to be avoided
                del data['odometer']
        return super(FleetVehicleLogServices, self).create(vals_list)

    @api.depends('vehicle_id')
    def _compute_purchaser_id(self):
        for service in self:
            service.purchaser_id = service.vehicle_id.driver_id

    def _expand_states(self, states, domain, order):
        return [key for key, dummy in self._fields['state'].selection]

```

## File: models\fleet_vehicle_model.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


FUEL_TYPES = [
    ('diesel', 'Diesel'),
    ('gasoline', 'Gasoline'),
    ('full_hybrid', 'Full Hybrid'),
    ('plug_in_hybrid_diesel', 'Plug-in Hybrid Diesel'),
    ('plug_in_hybrid_gasoline', 'Plug-in Hybrid Gasoline'),
    ('cng', 'CNG'),
    ('lpg', 'LPG'),
    ('hydrogen', 'Hydrogen'),
    ('electric', 'Electric'),
]

class FleetVehicleModel(models.Model):
    _name = 'fleet.vehicle.model'
    _description = 'Model of a vehicle'
    _order = 'name asc'

    name = fields.Char('Model name', required=True)
    brand_id = fields.Many2one('fleet.vehicle.model.brand', 'Manufacturer', required=True)
    category_id = fields.Many2one('fleet.vehicle.model.category', 'Category')
    vendors = fields.Many2many('res.partner', 'fleet_vehicle_model_vendors', 'model_id', 'partner_id', string='Vendors')
    image_128 = fields.Image(related='brand_id.image_128', readonly=True)
    active = fields.Boolean(default=True)
    vehicle_type = fields.Selection([('car', 'Car'), ('bike', 'Bike')], default='car', required=True)
    transmission = fields.Selection([('manual', 'Manual'), ('automatic', 'Automatic')], 'Transmission')
    vehicle_count = fields.Integer(compute='_compute_vehicle_count')
    model_year = fields.Integer()
    color = fields.Char()
    seats = fields.Integer(string='Seats Number')
    doors = fields.Integer(string='Doors Number')
    trailer_hook = fields.Boolean(default=False, string='Trailer Hitch')
    default_co2 = fields.Float('CO2 Emissions')
    co2_standard = fields.Char()
    default_fuel_type = fields.Selection(FUEL_TYPES, 'Fuel Type', default='electric')
    power = fields.Integer('Power')
    horsepower = fields.Integer()
    horsepower_tax = fields.Float('Horsepower Taxation')
    electric_assistance = fields.Boolean(default=False)

    def name_get(self):
        res = []
        for record in self:
            name = record.name
            if record.brand_id.name:
                name = record.brand_id.name + '/' + name
            res.append((record.id, name))
        return res

    def _compute_vehicle_count(self):
        group = self.env['fleet.vehicle']._read_group(
            [('model_id', 'in', self.ids)], ['id', 'model_id'], groupby='model_id', lazy=False,
        )
        count_by_model = {entry['model_id'][0]: entry['__count'] for entry in group}
        for model in self:
            model.vehicle_count = count_by_model.get(model.id, 0)

    def action_model_vehicle(self):
        self.ensure_one()
        view = {
            'type': 'ir.actions.act_window',
            'view_mode': 'kanban,tree,form',
            'res_model': 'fleet.vehicle',
            'name': _('Vehicles'),
            'context': {'search_default_model_id': self.id, 'default_model_id': self.id}
        }

        return view

```

## File: models\fleet_vehicle_model_brand.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class FleetVehicleModelBrand(models.Model):
    _name = 'fleet.vehicle.model.brand'
    _description = 'Brand of the vehicle'
    _order = 'name asc'

    name = fields.Char('Name', required=True)
    image_128 = fields.Image("Logo", max_width=128, max_height=128)
    model_count = fields.Integer(compute="_compute_model_count", string="", store=True)
    model_ids = fields.One2many('fleet.vehicle.model', 'brand_id')

    @api.depends('model_ids')
    def _compute_model_count(self):
        model_data = self.env['fleet.vehicle.model']._read_group([
            ('brand_id', 'in', self.ids),
        ], ['brand_id'], ['brand_id'])
        models_brand = {x['brand_id'][0]: x['brand_id_count'] for x in model_data}

        for record in self:
            record.model_count = models_brand.get(record.id, 0)

    def action_brand_model(self):
        self.ensure_one()
        view = {
            'type': 'ir.actions.act_window',
            'view_mode': 'tree,form',
            'res_model': 'fleet.vehicle.model',
            'name': 'Models',
            'context': {'search_default_brand_id': self.id, 'default_brand_id': self.id}
        }

        return view

```

## File: models\fleet_vehicle_model_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class FleetVehicleModelCategory(models.Model):
    _name = 'fleet.vehicle.model.category'
    _description = 'Category of the model'
    _order = 'sequence asc, id asc'

    _sql_constraints = [
        ('name_uniq', 'UNIQUE (name)', 'Category name must be unique')
    ]

    name = fields.Char(required=True)
    sequence = fields.Integer()

```

## File: models\fleet_vehicle_odometer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class FleetVehicleOdometer(models.Model):
    _name = 'fleet.vehicle.odometer'
    _description = 'Odometer log for a vehicle'
    _order = 'date desc'

    name = fields.Char(compute='_compute_vehicle_log_name', store=True)
    date = fields.Date(default=fields.Date.context_today)
    value = fields.Float('Odometer Value', group_operator="max")
    vehicle_id = fields.Many2one('fleet.vehicle', 'Vehicle', required=True)
    unit = fields.Selection(related='vehicle_id.odometer_unit', string="Unit", readonly=True)
    driver_id = fields.Many2one(related="vehicle_id.driver_id", string="Driver", readonly=False)

    @api.depends('vehicle_id', 'date')
    def _compute_vehicle_log_name(self):
        for record in self:
            name = record.vehicle_id.name
            if not name:
                name = str(record.date)
            elif record.date:
                name += ' / ' + str(record.date)
            record.name = name

    @api.onchange('vehicle_id')
    def _onchange_vehicle(self):
        if self.vehicle_id:
            self.unit = self.vehicle_id.odometer_unit

```

## File: models\fleet_vehicle_state.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class FleetVehicleState(models.Model):
    _name = 'fleet.vehicle.state'
    _order = 'sequence asc'
    _description = 'Vehicle Status'

    name = fields.Char(required=True, translate=True)
    sequence = fields.Integer()

    _sql_constraints = [('fleet_state_name_unique', 'unique(name)', 'State name already exists')]

```

## File: models\fleet_vehicle_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class FleetVehicleTag(models.Model):
    _name = 'fleet.vehicle.tag'
    _description = 'Vehicle Tag'

    name = fields.Char('Tag Name', required=True, translate=True)
    color = fields.Integer('Color')

    _sql_constraints = [('name_uniq', 'unique (name)', "Tag name already exists!")]

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = ['res.config.settings']

    delay_alert_contract = fields.Integer(string='Delay alert contract outdated', default=30, config_parameter='hr_fleet.delay_alert_contract')

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    plan_to_change_car = fields.Boolean('Plan To Change Car', default=False)
    plan_to_change_bike = fields.Boolean('Plan To Change Bike', default=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import fleet_service_type
from . import fleet_vehicle
from . import fleet_vehicle_assignation_log
from . import fleet_vehicle_log_contract
from . import fleet_vehicle_log_services
from . import fleet_vehicle_model
from . import fleet_vehicle_model_brand
from . import fleet_vehicle_model_category
from . import fleet_vehicle_odometer
from . import fleet_vehicle_state
from . import fleet_vehicle_tag
from . import res_config_settings
from . import res_partner

```

## File: report\fleet_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from psycopg2 import sql

from odoo import tools
from odoo import api, fields, models


class FleetReport(models.Model):
    _name = "fleet.vehicle.cost.report"
    _description = "Fleet Analysis Report"
    _auto = False
    _order = 'date_start desc'

    company_id = fields.Many2one('res.company', 'Company', readonly=True)
    vehicle_id = fields.Many2one('fleet.vehicle', 'Vehicle', readonly=True)
    name = fields.Char('Vehicle Name', readonly=True)
    driver_id = fields.Many2one('res.partner', 'Driver', readonly=True)
    fuel_type = fields.Char('Fuel', readonly=True)
    date_start = fields.Date('Date', readonly=True)
    vehicle_type = fields.Selection([('car', 'Car'), ('bike', 'Bike')], readonly=True)

    cost = fields.Float('Cost', readonly=True)
    cost_type = fields.Selection(string='Cost Type', selection=[
        ('contract', 'Contract'),
        ('service', 'Service')
    ], readonly=True)

    def init(self):
        query = """
WITH service_costs AS (
    SELECT
        ve.id AS vehicle_id,
        ve.company_id AS company_id,
        ve.name AS name,
        ve.driver_id AS driver_id,
        ve.fuel_type AS fuel_type,
        date(date_trunc('month', d)) AS date_start,
        vem.vehicle_type as vehicle_type,
        COALESCE(sum(se.amount), 0) AS
        COST,
        'service' AS cost_type
    FROM
        fleet_vehicle ve
    JOIN
        fleet_vehicle_model vem ON vem.id = ve.model_id
    CROSS JOIN generate_series((
            SELECT
                min(date)
                FROM fleet_vehicle_log_services), CURRENT_DATE + '1 month'::interval, '1 month') d
        LEFT JOIN fleet_vehicle_log_services se ON se.vehicle_id = ve.id
            AND date_trunc('month', se.date) = date_trunc('month', d)
    WHERE
        ve.active AND se.active AND se.state != 'cancelled'
    GROUP BY
        ve.id,
        ve.company_id,
        vem.vehicle_type,
        ve.name,
        date_start,
        d
    ORDER BY
        ve.id,
        date_start
),
contract_costs AS (
    SELECT
        ve.id AS vehicle_id,
        ve.company_id AS company_id,
        ve.name AS name,
        ve.driver_id AS driver_id,
        ve.fuel_type AS fuel_type,
        date(date_trunc('month', d)) AS date_start,
        vem.vehicle_type as vehicle_type,
        (COALESCE(sum(co.amount), 0) + COALESCE(sum(cod.cost_generated * extract(day FROM least (date_trunc('month', d) + interval '1 month', cod.expiration_date) - greatest (date_trunc('month', d), cod.start_date))), 0) + COALESCE(sum(com.cost_generated), 0) + COALESCE(sum(coy.cost_generated), 0)) AS
        COST,
        'contract' AS cost_type
    FROM
        fleet_vehicle ve
    JOIN
        fleet_vehicle_model vem ON vem.id = ve.model_id
    CROSS JOIN generate_series((
            SELECT
                min(acquisition_date)
                FROM fleet_vehicle), CURRENT_DATE + '1 month'::interval, '1 month') d
        LEFT JOIN fleet_vehicle_log_contract co ON co.vehicle_id = ve.id
            AND date_trunc('month', co.date) = date_trunc('month', d)
        LEFT JOIN fleet_vehicle_log_contract cod ON cod.vehicle_id = ve.id
            AND date_trunc('month', cod.start_date) <= date_trunc('month', d)
            AND date_trunc('month', cod.expiration_date) >= date_trunc('month', d)
            AND cod.cost_frequency = 'daily'
    LEFT JOIN fleet_vehicle_log_contract com ON com.vehicle_id = ve.id
        AND date_trunc('month', com.start_date) <= date_trunc('month', d)
        AND date_trunc('month', com.expiration_date) >= date_trunc('month', d)
        AND com.cost_frequency = 'monthly'
    LEFT JOIN fleet_vehicle_log_contract coy ON coy.vehicle_id = ve.id
        AND d BETWEEN coy.start_date and coy.expiration_date
        AND date_part('month', coy.date) = date_part('month', d)
        AND coy.cost_frequency = 'yearly'
    WHERE
        ve.active
    GROUP BY
        ve.id,
        ve.company_id,
        vem.vehicle_type,
        ve.name,
        date_start,
        d
    ORDER BY
        ve.id,
        date_start
)
SELECT row_number() OVER (ORDER BY vehicle_id ASC) as id,
    company_id,
    vehicle_id,
    name,
    driver_id,
    fuel_type,
    date_start,
    vehicle_type,
    COST,
    cost_type
FROM (
    SELECT
        company_id,
        vehicle_id,
        name,
        driver_id,
        fuel_type,
        date_start,
        vehicle_type,
        COST,
        'service' as cost_type
    FROM
        service_costs sc
    UNION ALL (
        SELECT
            company_id,
            vehicle_id,
            name,
            driver_id,
            fuel_type,
            date_start,
            vehicle_type,
            COST,
            'contract' as cost_type
        FROM
            contract_costs cc)
) c
"""
        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute(
            sql.SQL("""CREATE or REPLACE VIEW {} as ({})""").format(
                sql.Identifier(self._table),
                sql.SQL(query)
            ))

```

## File: report\__init__.py

```python
from . import fleet_report

```

## File: security\fleet_security.xml

```xml
<?xml version="1.0" ?>
<odoo>
        <record id="module_fleet_category" model="ir.module.category">
            <field name="name">Fleet</field>
            <field name="sequence">17</field>
        </record>
        <record id="fleet_group_user" model="res.groups">
            <field name="name">Officer : Manage all vehicles</field>
            <field name="category_id" ref="base.module_category_human_resources_fleet"/>
            <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
        </record>
        <record id="fleet_group_manager" model="res.groups">
            <field name="name">Administrator</field>
            <field name="implied_ids" eval="[(4, ref('fleet_group_user'))]"/>
            <field name="category_id" ref="base.module_category_human_resources_fleet"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

    <data noupdate="1">
        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('fleet.fleet_group_manager'))]"/>
        </record>
        <record id="fleet_rule_contract_visibility_user" model="ir.rule">
            <field name="name">User can only see his/her contracts</field>
            <field name="model_id" ref="model_fleet_vehicle_log_contract"/>
            <field name="groups" eval="[(4, ref('fleet_group_user'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
            <field name="domain_force">[('vehicle_id.driver_id','=',user.partner_id.id)]</field>
        </record>
        <record id="fleet_rule_service_visibility_user" model="ir.rule">
            <field name="name">User can only see his/her vehicle's services</field>
            <field name="model_id" ref="model_fleet_vehicle_log_services"/>
            <field name="groups" eval="[(4, ref('fleet_group_user'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
            <field name="domain_force">[('vehicle_id.driver_id','=',user.partner_id.id)]</field>
        </record>
        <record id="fleet_rule_odometer_visibility_user" model="ir.rule">
            <field name="name">User can only see his/her vehicle's odometer</field>
            <field name="model_id" ref="model_fleet_vehicle_odometer"/>
            <field name="groups" eval="[(4, ref('fleet_group_user'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="True"/>
            <field name="perm_unlink" eval="True"/>
            <field name="domain_force">[('vehicle_id.driver_id','=',user.partner_id.id)]</field>
        </record>
        <record id="fleet_rule_vehicle_visibility_user" model="ir.rule">
            <field name="name">User can only see his/her vehicle</field>
            <field name="model_id" ref="model_fleet_vehicle"/>
            <field name="groups" eval="[(4, ref('fleet_group_user'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
            <field name="domain_force">[('driver_id','=',user.partner_id.id)]</field>
        </record>
        <record id="fleet_rule_contract_visibility_manager" model="ir.rule">
            <field name="name">Administrator has all rights on vehicle's contracts</field>
            <field name="model_id" ref="model_fleet_vehicle_log_contract"/>
            <field name="groups" eval="[Command.link(ref('fleet_group_manager'))]"/>
        </record>
        <record id="fleet_rule_service_visibility_manager" model="ir.rule">
            <field name="name">Administrator has all rights on vehicle's services</field>
            <field name="model_id" ref="model_fleet_vehicle_log_services"/>
            <field name="groups" eval="[Command.link(ref('fleet_group_manager'))]"/>
        </record>
        <record id="fleet_rule_odometer_visibility_manager" model="ir.rule">
            <field name="name">Administrator has all rights on vehicle's vehicle's odometer</field>
            <field name="model_id" ref="model_fleet_vehicle_odometer"/>
            <field name="groups" eval="[Command.link(ref('fleet_group_manager'))]"/>
        </record>
        <record id="fleet_rule_vehicle_visibility_manager" model="ir.rule">
            <field name="name">Administrator has all rights on vehicle</field>
            <field name="model_id" ref="model_fleet_vehicle"/>
            <field name="groups" eval="[Command.link(ref('fleet_group_manager'))]"/>
        </record>
        <record id="ir_rule_fleet_vehicle" model="ir.rule">
            <field name="name">Fleet vehicle: Multi Company</field>
            <field name="model_id" ref="model_fleet_vehicle"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>
        <record id="ir_rule_fleet_vehicle_log_contract" model="ir.rule">
            <field name="name">Fleet vehicle log contract: Multi Company</field>
            <field name="model_id" ref="model_fleet_vehicle_log_contract"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>
        <record id="ir_rule_fleet_report" model="ir.rule">
            <field name="name">Costs Analysis: Multi Company</field>
            <field name="model_id" ref="model_fleet_vehicle_cost_report"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>
        <record id="ir_rule_fleet_odometer" model="ir.rule">
            <field name="name">Fleet odometer: Multi Company</field>
            <field name="model_id" ref="model_fleet_vehicle_odometer"/>
            <field name="global" eval="True"/>
            <field name="domain_force">[('vehicle_id.company_id', 'in', company_ids + [False])]</field>
        </record>
        <record id="ir_rule_fleet_log_services" model="ir.rule">
            <field name="name">Fleet log services: Multi Company</field>
            <field name="model_id" ref="model_fleet_vehicle_log_services"/>
            <field name="global" eval="True"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
fleet_vehicle_model_access_right_user,fleet_vehicle_model_access_right,model_fleet_vehicle_model,fleet_group_user,1,0,0,0
fleet_vehicle_tag_access_right_user,fleet_vehicle_tag_access_right,model_fleet_vehicle_tag,fleet_group_user,1,0,0,0
fleet_vehicle_state_access_right_user,fleet_vehicle_state_access_right,model_fleet_vehicle_state,fleet_group_user,1,0,0,0
fleet_vehicle_model_brand_access_right_user,fleet_vehicle_model_brand_access_right,model_fleet_vehicle_model_brand,fleet_group_user,1,0,0,0
fleet_vehicle_model_brand_category_right_user,fleet_vehicle_model_category_access_right,model_fleet_vehicle_model_category,fleet_group_user,1,0,0,0
fleet_vehicle_access_right_user,fleet_vehicle_access_right,model_fleet_vehicle,fleet_group_user,1,1,0,0
fleet_vehicle_log_services_access_right_user,fleet_vehicle_log_services_access_right,model_fleet_vehicle_log_services,fleet_group_user,1,0,0,0
fleet_vehicle_log_contract_access_right_user,fleet_vehicle_log_contract_access_right,model_fleet_vehicle_log_contract,fleet_group_user,1,0,0,0
fleet_service_type_access_right_user,fleet_service_type_access_right,model_fleet_service_type,fleet_group_user,1,0,0,0
fleet_vehicle_model_access_right,fleet_vehicle_model_access_right,model_fleet_vehicle_model,fleet_group_manager,1,1,1,1
fleet_vehicle_tag_access_right,fleet_vehicle_tag_access_right,model_fleet_vehicle_tag,fleet_group_manager,1,1,1,1
fleet_vehicle_state_access_right,fleet_vehicle_state_access_right,model_fleet_vehicle_state,fleet_group_manager,1,1,1,1
fleet_vehicle_odometer_access_right,fleet_vehicle_odometer_access_right,model_fleet_vehicle_odometer,fleet_group_user,1,1,1,1
fleet_vehicle_model_brand_access_right,fleet_vehicle_model_brand_access_right,model_fleet_vehicle_model_brand,fleet_group_manager,1,1,1,1
fleet_vehicle_model_category_access_right,fleet_vehicle_model_brand_category_right,model_fleet_vehicle_model_category,fleet_group_manager,1,1,1,1
fleet_vehicle_access_right,fleet_vehicle_access_right,model_fleet_vehicle,fleet_group_manager,1,1,1,1
fleet_vehicle_log_services_access_right,fleet_vehicle_log_services_access_right,model_fleet_vehicle_log_services,fleet_group_manager,1,1,1,1
fleet_vehicle_log_contract_access_right,fleet_vehicle_log_contract_access_right,model_fleet_vehicle_log_contract,fleet_group_manager,1,1,1,1
fleet_service_type_access_right,fleet_service_type_access_right,model_fleet_service_type,fleet_group_manager,1,1,1,1
access_mail_activity_type_fleet_manager,mail.activity.type.fleet.manager,mail.model_mail_activity_type,fleet.fleet_group_manager,1,1,1,1
access_fleet_vehicle_assignation_log_fleet_group_user,fleet_vehicle_assignation_log fleet_group_user,fleet.model_fleet_vehicle_assignation_log,fleet.fleet_group_user,1,1,1,1
access_fleet_report_manager,fleet_vehicle_cost_report_access_right,model_fleet_vehicle_cost_report,fleet_group_manager,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="98.162%" x2="0%" y1="1.838%" y2="100%">
            <stop offset="0%" stop-color="#797DA5"/>
            <stop offset="50.799%" stop-color="#6D7194"/>
            <stop offset="100%" stop-color="#626584"/>
        </linearGradient>
        <path id="icon-d" d="M54.8560384,29.8385417 L50.3951823,29.8385417 L49.7560221,28.1341146 C49.0079753,26.1392415 47.6869303,24.4434408 45.9358724,23.2298991 C44.1848145,22.0163574 42.1330566,21.375 40.0026042,21.375 L29.9973958,21.375 C27.866862,21.375 25.8151855,22.0164388 24.0641276,23.2298991 C22.3129883,24.4434408 20.9920247,26.1392415 20.2439779,28.1341146 L19.6048177,29.8385417 L15.1439616,29.8385417 C14.4773763,29.8385417 14.0067546,30.4915365 14.2175293,31.1239421 L14.868571,33.0770671 C15.0015005,33.4758244 15.3746728,33.7447878 15.7950033,33.7447917 L18.139974,33.7447917 L18.1357422,33.7561849 C16.5447591,34.6475423 15.46875,36.3487142 15.46875,38.3020833 L15.46875,42.2083333 C15.46875,43.5287272 15.9610189,44.7334798 16.7708333,45.6514486 L16.7708333,50.671875 C16.7708333,51.7505697 17.6452637,52.625 18.7239583,52.625 L22.6302083,52.625 C23.708903,52.625 24.5833333,51.7505697 24.5833333,50.671875 L24.5833333,47.4166667 L45.4166667,47.4166667 L45.4166667,50.671875 C45.4166667,51.7505697 46.291097,52.625 47.3697917,52.625 L51.2760417,52.625 C52.3547363,52.625 53.2291667,51.7505697 53.2291667,50.671875 L53.2291667,45.6514486 C54.0389811,44.7333984 54.53125,43.5286458 54.53125,42.2083333 L54.53125,38.3020833 C54.53125,36.3487142 53.4552409,34.6475423 51.8642578,33.7561849 L51.860026,33.7447917 L54.2049967,33.7447917 C54.6253272,33.7447878 54.9984995,33.4758244 55.131429,33.0770671 L55.7824707,31.1239421 C55.9932454,30.4915365 55.5226237,29.8385417 54.8560384,29.8385417 Z M25.1206868,29.9628906 C25.8787435,27.9414876 27.8385417,26.5833333 29.9973958,26.5833333 L40.0026042,26.5833333 C42.1614583,26.5833333 44.1212565,27.9414876 44.8793132,29.9628906 L46.0533854,33.09375 L23.9466146,33.09375 L25.1206868,29.9628906 Z M21.328125,42.859375 C19.8898926,42.859375 18.7239583,41.6934408 18.7239583,40.2552083 C18.7239583,38.8169759 19.8898926,37.6510417 21.328125,37.6510417 C22.7663574,37.6510417 25.234375,40.1190592 25.234375,41.5572917 C25.234375,42.9955241 22.7663574,42.859375 21.328125,42.859375 Z M48.671875,42.859375 C47.2336426,42.859375 44.765625,42.9955241 44.765625,41.5572917 C44.765625,40.1190592 47.2336426,37.6510417 48.671875,37.6510417 C50.1101074,37.6510417 51.2760417,38.8169759 51.2760417,40.2552083 C51.2760417,41.6934408 50.1101074,42.859375 48.671875,42.859375 Z"/>
        <path id="icon-e" d="M54.8560384,27.8385417 L50.3951823,27.8385417 L49.7560221,26.1341146 C49.0079753,24.1392415 47.6869303,22.4434408 45.9358724,21.2298991 C44.1848145,20.0163574 42.1330566,19.375 40.0026042,19.375 L29.9973958,19.375 C27.866862,19.375 25.8151855,20.0164388 24.0641276,21.2298991 C22.3129883,22.4434408 20.9920247,24.1392415 20.2439779,26.1341146 L19.6048177,27.8385417 L15.1439616,27.8385417 C14.4773763,27.8385417 14.0067546,28.4915365 14.2175293,29.1239421 L14.868571,31.0770671 C15.0015005,31.4758244 15.3746728,31.7447878 15.7950033,31.7447917 L18.139974,31.7447917 L18.1357422,31.7561849 C16.5447591,32.6475423 15.46875,34.3487142 15.46875,36.3020833 L15.46875,40.2083333 C15.46875,41.5287272 15.9610189,42.7334798 16.7708333,43.6514486 L16.7708333,48.671875 C16.7708333,49.7505697 17.6452637,50.625 18.7239583,50.625 L22.6302083,50.625 C23.708903,50.625 24.5833333,49.7505697 24.5833333,48.671875 L24.5833333,45.4166667 L45.4166667,45.4166667 L45.4166667,48.671875 C45.4166667,49.7505697 46.291097,50.625 47.3697917,50.625 L51.2760417,50.625 C52.3547363,50.625 53.2291667,49.7505697 53.2291667,48.671875 L53.2291667,43.6514486 C54.0389811,42.7333984 54.53125,41.5286458 54.53125,40.2083333 L54.53125,36.3020833 C54.53125,34.3487142 53.4552409,32.6475423 51.8642578,31.7561849 L51.860026,31.7447917 L54.2049967,31.7447917 C54.6253272,31.7447878 54.9984995,31.4758244 55.131429,31.0770671 L55.7824707,29.1239421 C55.9932454,28.4915365 55.5226237,27.8385417 54.8560384,27.8385417 Z M25.1206868,27.9628906 C25.8787435,25.9414876 27.8385417,24.5833333 29.9973958,24.5833333 L40.0026042,24.5833333 C42.1614583,24.5833333 44.1212565,25.9414876 44.8793132,27.9628906 L46.0533854,31.09375 L23.9466146,31.09375 L25.1206868,27.9628906 Z M21.328125,40.859375 C19.8898926,40.859375 18.7239583,39.6934408 18.7239583,38.2552083 C18.7239583,36.8169759 19.8898926,35.6510417 21.328125,35.6510417 C22.7663574,35.6510417 25.234375,38.1190592 25.234375,39.5572917 C25.234375,40.9955241 22.7663574,40.859375 21.328125,40.859375 Z M48.671875,40.859375 C47.2336426,40.859375 44.765625,40.9955241 44.765625,39.5572917 C44.765625,38.1190592 47.2336426,35.6510417 48.671875,35.6510417 C50.1101074,35.6510417 51.2760417,36.8169759 51.2760417,38.2552083 C51.2760417,39.6934408 50.1101074,40.859375 48.671875,40.859375 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M4,48 C2,48 -7.05642142e-15,47.8509317 4.59701721e-17,43.826087 L0,18.0539143 L14.3290638,6.33400231 L16.5273341,6.02061955 L23.2330679,0 L44,0 L53,13.5652174 L53,27.5773673 L37.5658729,48 L4,48 Z" opacity=".324" transform="translate(0 22)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#icon-d"/>
            <use fill="#FFF" fill-rule="nonzero" xlink:href="#icon-e"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\fleet_form.js

```javascript
/** @odoo-module **/

import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { registry } from "@web/core/registry";
import { FormController } from "@web/views/form/form_controller";
import { formView } from "@web/views/form/form_view";

export class FleetFormController extends FormController {
    /**
     * @override
     **/
    getActionMenuItems() {
        const menuItems = super.getActionMenuItems();
        const archiveAction = menuItems.other.find((item) => item.key === "archive");
        if (archiveAction) {
            archiveAction.callback = () => {
                const dialogProps = {
                    body: this.env._t(
                        "Every service and contract of this vehicle will be considered as archived. Are you sure that you want to archive this record?"
                    ),
                    confirm: () => this.model.root.archive(),
                    cancel: () => {},
                };
                this.dialogService.add(ConfirmationDialog, dialogProps);
            };
        }
        return menuItems;
    }
}

export const fleetFormView = {
    ...formView,
    Controller: FleetFormController,
};

registry.category("views").add("fleet_form", fleetFormView);

```

## File: views\fleet_board_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fleet_costs_report_view_search" model="ir.ui.view">
        <field name="name">fleet.vehicle.cost.view.search</field>
        <field name="model">fleet.vehicle.cost.report</field>
        <field name="arch" type="xml">
            <search string="Fleet Costs Analysis">
                <field name="name" filter_domain="[('name', 'ilike', self)]"/>
                <field name="driver_id" filter_domain="[('driver_id', 'ilike', self)]"/>
                <field name="date_start"/>
                <filter string="Service" name="service" domain="[('cost_type', '=', 'service')]"/>
                <filter string="Contract" name="contract" domain="[('cost_type', '=', 'contract')]"/>
                <separator/>
                <filter name="filter_date_start" date="date_start" default_period="this_year"/>
                <group expand="1" string="Group By">
                    <filter string="Vehicle" name="vehicle" context="{'group_by':'vehicle_id'}"/>
                    <filter string="Driver" name="driver" context="{'group_by':'driver_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="fleet_costs_report_view_pivot" model="ir.ui.view">
        <field name="name">fleet.vehicle.cost.view.pivot</field>
        <field name="model">fleet.vehicle.cost.report</field>
        <field name="arch" type="xml">
            <pivot sample="1">
                <field name="date_start" type="col" interval="year" />
                <field name="cost_type" type="col" />
                <field name="vehicle_id" type="row" />
                <field name="cost" type="measure" />
            </pivot>
        </field>
    </record>

    <record id="fleet_costs_report_view_graph" model="ir.ui.view">
        <field name="name">fleet.vehicle.cost.view.graph</field>
        <field name="model">fleet.vehicle.cost.report</field>
        <field name="arch" type="xml">
            <graph string="Fleet Costs Analysis" sample="1">
                <field name="date_start" interval="month"/>
                <field name="cost_type"/>
                <field name="cost" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="fleet_vechicle_costs_report_view_tree" model="ir.ui.view">
        <field name="name">fleet.vehicle.cost.report.view.tree</field>
        <field name="model">fleet.vehicle.cost.report</field>
        <field name="arch" type="xml">
            <tree string="Fleet Costs Analysis">
                <field name="name"/>
                <field name="driver_id" optional="show"/>
                <field name="fuel_type" optional="hide"/>
                <field name="date_start" optional="show"/>
                <field name="cost" optional="show" sum="Sum of Cost"/>
                <field name="cost_type" optional="show"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

 <record id="fleet_costs_reporting_action" model="ir.actions.act_window">
      <field name="name">Costs Analysis</field>
      <field name="res_model">fleet.vehicle.cost.report</field>
      <field name="view_mode">graph,pivot</field>
      <field name="view_id"></field>
      <field name="context" eval="{'search_default_filter_date_start': 1}"/>
      <field name="search_view_id" ref="fleet.fleet_costs_report_view_search"/>
      <field name="help" type="html">
        <p class="o_view_nocontent_empty_folder">
          No data for analysis
        </p><p>
          Manage efficiently your different effective vehicles Costs with Odoo.
        </p>
      </field>
    </record>
    <menuitem name="Reporting" parent="menu_root" id="menu_fleet_reporting" sequence="99" groups="fleet_group_manager"/>
    <menuitem id="menu_fleet_reporting_costs"
              name="Costs"
              parent="menu_fleet_reporting"
              action="fleet_costs_reporting_action"
              sequence="1"
              groups="fleet_group_manager"/>
</odoo>

```

## File: views\fleet_vehicle_cost_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id='fleet_vehicle_log_contract_view_form' model='ir.ui.view'>
        <field name="name">fleet.vehicle.log_contract.form</field>
        <field name="model">fleet.vehicle.log.contract</field>
        <field name="arch" type="xml">
            <form string="Contract logs">
                <field name="company_id" invisible="1"/>
                <header>
                    <button name="action_open" states="futur" type="object" string="Start Contract" class="oe_highlight" groups="fleet.fleet_group_manager"/>
                    <button name="action_close" states="futur" type="object" string="Cancel" groups="fleet.fleet_group_manager"/>
                    <button name="action_close" states="open,expired,futur" type="object" string="Close Contract" groups="fleet.fleet_group_manager"/>
                    <button name="action_draft" states="closed" type="object" string="Reset To Draft" groups="fleet.fleet_group_manager"/>
                    <field name="state" widget="statusbar" />
                </header>
                <sheet>
                    <field name="active" invisible="1"/>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <field name="currency_id" invisible="1"/>
                    <div class="oe_title">
                        <h1><field name="name"/></h1>
                    </div>
                    <group string="Information" col="2">
                        <group>
                            <field name="ins_ref"/>
                            <field name="cost_subtype_id"/>
                            <field name="insurer_id"/>
                            <field name="service_ids" widget="many2many_tags"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <group>
                            <field name="start_date"/>
                            <field name="expiration_date" attrs="{'required': [('cost_frequency', '!=', 'no')]}"/>
                            <field name="user_id"/>
                            <field name="purchaser_id" invisible="1"/>
                        </group>
                    </group>
                    <separator string="Vehicle"/>
                    <group col="2">
                        <group col="1">
                            <field name="vehicle_id"/>
                        </group>
                        <group col="2">
                            <field name="purchaser_id"/>
                        </group>
                    </group>
                    <separator string="Cost"/>
                    <group col="2">
                        <group>
                            <field name="amount" string="Activation Cost" help="Cost that is paid only once at the creation of the contract" widget="monetary" class="w-25"/>
                            <label for="cost_generated"/>
                            <div class="o_row">
                                <span class="w-25">
                                    <field name="cost_generated" attrs="{'invisible': [('cost_frequency','=','no')]}" widget="monetary"/>
                                </span>
                                <field name="cost_frequency"/>
                            </div>
                        </group>
                        <group>
                            <field name="date"/>
                        </group>
                    </group>
                    <separator string="Terms and Conditions"/>
                    <field name="notes" nolabel="1" placeholder="Write here all other information relative to this contract" />
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id='fleet_vehicle_log_contract_view_tree' model='ir.ui.view'>
        <field name="name">fleet.vehicle.log.contract.tree</field>
        <field name="model">fleet.vehicle.log.contract</field>
        <field name="arch" type="xml">
            <tree string="Contract logs"
                decoration-warning="expires_today"
                decoration-danger="days_left==0 and not expires_today"
                decoration-muted="state=='closed'"
                default_order="expiration_date"
                sample="1">
                <field name="active" invisible="1"/>
                <field name="expires_today" invisible="1"/>
                <field name="name" class="fw-bold" />
                <field name="start_date" />
                <field name="expiration_date" widget="remaining_days"/>
                <field name="days_left" invisible="1"/>
                <field name="vehicle_id"/>
                <field name="insurer_id" />
                <field name="purchaser_id" widget="many2one_avatar"/>
                <field name="cost_generated" widget="monetary"/>
                <field name="currency_id" invisible="1"/>
                <field name="cost_frequency"/>
                <field name="state" widget="badge" decoration-info="state == 'open'" decoration-danger="state == 'expired'" />
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record id='fleet_vehicle_log_contract_view_kanban' model='ir.ui.view'>
        <field name="name">fleet.vehicle.log.contract.kanban</field>
        <field name="model">fleet.vehicle.log.contract</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1">
                <field name="activity_state"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div>
                                <strong>
                                    <field name="vehicle_id" widget="res_partner_many2one"/>
                                    <span class="float-end badge text-bg-secondary">
                                        <field name="state"/>
                                    </span>
                                </strong>
                            </div>
                            <div>
                                <t t-if="luxon.DateTime.fromISO(record.expiration_date.raw_value) &lt; luxon.DateTime.local()" t-set="expiration_class" t-value="'oe_kanban_text_red'"/>
                                <span t-att-class="expiration_class"><field name="start_date"/> - <field name="expiration_date"/></span>
                            </div>
                            <div>
                                <field name="insurer_id" widget="res_partner_many2one"/>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="fleet_vehicle_log_contract_view_graph" model="ir.ui.view">
       <field name="name">fleet.vehicle.log.contract.graph</field>
       <field name="model">fleet.vehicle.log.contract</field>
       <field name="arch" type="xml">
            <graph string="Contract Costs Per Month" sample="1">
                <field name="date"/>
                <field name="vehicle_id"/>
                <field name="amount" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="fleet_vehicle_log_contract_view_search" model="ir.ui.view">
        <field name="name">fleet.vehicle.log.contract.search</field>
        <field name="model">fleet.vehicle.log.contract</field>
        <field name="arch" type="xml">
            <search string="Vehicles Contracts">
                <field name="vehicle_id" string="Vehicle" filter_domain="[('vehicle_id.name','ilike', self)]"/>
                <field name="purchaser_id" string="Driver" filter_domain="[('purchaser_id','child_of', self)]"/>
                <field name="insurer_id" string="Vendor" filter_domain="[('insurer_id','child_of', self)]"/>
                <filter string="In Progress" name="open" domain="[('state', '=', 'open')]"/>
                <filter string="Expired" name="expired" domain="[('state', '=', 'expired')]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <group expand="0" string="Group By">
                    <filter string="Vehicle" name="vehicle" context="{'group_by': 'vehicle_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="fleet_vehicle_log_contract_view_activity" model="ir.ui.view">
        <field name="name">fleet.vehicle.log.contract.activity</field>
        <field name="model">fleet.vehicle.log.contract</field>
        <field name="arch" type="xml">
            <activity string="Vehicles Contracts">
                <field name="purchaser_id"/>
                <templates>
                    <div t-name="activity-box">
                        <img t-att-src="activity_image('res.partner', 'avatar_128', record.purchaser_id.raw_value)" t-att-title="record.purchaser_id.value" t-att-alt="record.purchaser_id.value"/>
                        <div>
                            <field name="vehicle_id" display="full"/>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record id="fleet_vehicle_log_contract_view_pivot" model="ir.ui.view">
       <field name="model">fleet.vehicle.log.contract</field>
       <field name="arch" type="xml">
            <pivot>
                <field name="expiration_date" type="col" />
                <field name="cost_subtype_id" type="row" />
                <field name="vehicle_id" type="row" />
            </pivot>
        </field>
    </record>

    <record id='fleet_vehicle_log_contract_action' model='ir.actions.act_window'>
        <field name="name">Contracts</field>
        <field name="res_model">fleet.vehicle.log.contract</field>
        <field name="view_mode">tree,kanban,form,graph,pivot,activity</field>
        <field name="context">{'search_default_open': 1}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new contract
          </p><p>
            Manage all your contracts (leasing, insurances, etc.) with
            their related services, costs. Odoo will automatically warn
            you when some contracts have to be renewed.
          </p><p>
            Each contract (e.g.: leasing) may include several services
            (reparation, insurances, periodic maintenance).
          </p>
        </field>
    </record>

    <menuitem action="fleet_vehicle_log_contract_action" parent="fleet_vehicles" id="fleet_vehicle_log_contract_menu" groups="fleet_group_user" sequence="2"/>

    <record id='fleet_vehicle_log_services_view_form' model='ir.ui.view'>
        <field name="name">fleet.vehicle.log.services.form</field>
        <field name="model">fleet.vehicle.log.services</field>
        <field name="arch" type="xml">
            <form string="Services Logs">
                <field name="active" invisible="1" />
                <field name="currency_id" invisible="1" />
                <header>
                    <field name="state" widget="statusbar" options="{'clickable': '1'}"/>
                </header>
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <group col="2">
                        <group>
                            <field name="description" />
                            <field name="service_type_id" />
                            <field name="date" />
                            <field name="amount" widget="monetary"/>
                            <field name="vendor_id"/>
                        </group>
                        <group>
                            <field name="vehicle_id"/>
                            <field name="purchaser_id"/>
                            <label for="odometer"/>
                            <div class="o_row">
                                <field name="odometer" class="w-25"/>
                                <field name="odometer_unit" class="ps-1 ps-sm-0"/>
                            </div>
                        </group>
                    </group>
                    <separator string="Notes"/>
                    <field nolabel="1" name="notes" placeholder="Write here any other information related to the service completed."/>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id='fleet_vehicle_log_services_view_tree' model='ir.ui.view'>
        <field name="name">fleet.vehicle.log.services.tree</field>
        <field name="model">fleet.vehicle.log.services</field>
        <field name="arch" type="xml">
            <tree string="Services Logs" multi_edit="1" expand="1">
                <field name="date" readonly="1" />
                <field name="description" />
                <field name="service_type_id" />
                <field name="vehicle_id" readonly="1"/>
                <field name="purchaser_id" readonly="1" widget="many2one_avatar"/>
                <field name="vendor_id" optional="show" />
                <field name="inv_ref" invisible="1" />
                <field name="notes" optional="show" />
                <field name="amount" sum="Total" widget="monetary"/>
                <field name="currency_id" invisible="1"/>
                <field name="state" readonly="1" widget="badge" decoration-success="state == 'done'" decoration-warning="state == 'new'"  decoration-info="state == 'running'" />
            </tree>
        </field>
    </record>

    <record id='fleet_vehicle_log_services_view_kanban' model='ir.ui.view'>
        <field name="name">fleet.vehicle.log.services.kanban</field>
        <field name="model">fleet.vehicle.log.services</field>
        <field name="arch" type="xml">
            <kanban default_group_by="state">
                <field name="currency_id"/>
                <field name="activity_ids"/>
                <field name="activity_state"/>
                <field name="purchaser_id"/>
                <field name="vendor_id"/>
                <field name="amount"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click container" class="o_kanban_record_has_image_fill">
                            <div class="oe_kanban_details">
                                <div class="o_kanban_record_top">
                                    <img t-att-src="kanban_image('fleet.vehicle', 'image_128', record.vehicle_id.raw_value)" t-att-alt="record.vehicle_id.value" class="o_image_24_cover float-start"/>
                                    <div class="o_kanban_record_headings ps-2 pe-2">
                                        <div class="text-truncate o_kanban_record_title">
                                            <strong>
                                                <field name="vehicle_id"/>
                                                <span t-attf-class="float-end badge #{['todo', 'running'].indexOf(record.state.raw_value) > -1 ? 'text-bg-secondary' : ['cancelled'].indexOf(record.state.raw_value) > -1 ? 'text-bg-danger' : 'text-bg-success'}">
                                                    <field name="state"/>
                                                </span>
                                            </strong>
                                        </div>
                                        <div class="text-truncate">
                                            <em><field name="service_type_id"/></em>
                                        </div>
                                    </div>
                                </div>
                                <div class="text-truncate">
                                    <field name="purchaser_id"/>
                                    <span class="float-end"><field name="date"/></span>
                                </div>
                                <div class="text-truncate">
                                    <field name="vendor_id"/>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <field name="amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="fleet_vehicle_log_services_view_graph" model="ir.ui.view">
       <field name="name">fleet.vehicle.log.services.graph</field>
       <field name="model">fleet.vehicle.log.services</field>
       <field name="arch" type="xml">
            <graph string="Services Costs Per Month" sample="1">
                <field name="date"/>
                <field name="vehicle_id"/>
                <field name="amount" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="fleet_vehicle_log_services_view_activity" model="ir.ui.view">
        <field name="model">fleet.vehicle.log.services</field>
        <field name="arch" type="xml">
            <activity string="Services">
                <templates>
                    <div t-name="activity-box">
                        <img t-att-src="activity_image('fleet.vehicle', 'image_128', record.vehicle_id.raw_value)" role="img" t-att-title="record.vehicle_id.value" t-att-alt="record.vehicle_id.value"/>
                        <div>
                            <field name="vehicle_id"/>
                            <t t-if="record.description.raw_value">
                                : <field name="description"/>
                            </t>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record id="fleet_vehicle_log_services_view_pivot" model="ir.ui.view">
       <field name="model">fleet.vehicle.log.services</field>
       <field name="arch" type="xml">
            <pivot>
                <field name="currency_id" invisible="1" />
                <field name="service_type_id" type="col" />
                <field name="vendor_id" type="row" />
                <field name="vehicle_id" type="row" />
                <field name="amount" type="measure" />
            </pivot>
        </field>
    </record>

    <record id='fleet_vehicle_log_services_view_search' model='ir.ui.view'>
        <field name="name">fleet.vehicle.log.services.search</field>
        <field name="model">fleet.vehicle.log.services</field>
        <field name="arch" type="xml">
            <search string="Services Logs" >
                <field name="vehicle_id"/>
                <field name="service_type_id"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                <group expand="1" string="Group By">
                    <filter string="Service Type" name="groupby_service_type_id" context="{'group_by': 'service_type_id'}"/>
                    <filter string="Fleet Manager" name="groupby_manager_id" context="{'group_by': 'manager_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id='fleet_vehicle_log_services_action' model='ir.actions.act_window'>
        <field name="name">Services</field>
        <field name="res_model">fleet.vehicle.log.services</field>
        <field name="view_mode">tree,kanban,form,graph,pivot,activity</field>
        <field name="context">{'search_default_groupby_service_type_id': 1}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new service entry
          </p><p>
            Track all the services done on your vehicle.
            Services can be of many types: occasional repair, fixed maintenance, etc.
          </p>
        </field>
    </record>

    <menuitem action="fleet_vehicle_log_services_action" parent="fleet_vehicles" id="fleet_vehicle_log_services_menu" groups="fleet_group_user" sequence="3"/>

</odoo>

```

## File: views\fleet_vehicle_model_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id='fleet_vehicle_model_view_form' model='ir.ui.view'>
        <field name="name">fleet.vehicle.model.form</field>
        <field name="model">fleet.vehicle.model</field>
        <field name="arch" type="xml">
            <form string="Model">
                <sheet>
                    <widget name="web_ribbon" text="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_model_vehicle" type="object" icon="fa-car" class="oe_stat_button"
                            attrs="{'invisible': [('vehicle_count', '=', 0)]}">
                            <field name="vehicle_count" widget="statinfo" string="Vehicles"/>
                        </button>
                    </div>
                    <field name="image_128" widget='image' class="oe_avatar"/>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Model S"/>
                        </h1>
                        <label for="brand_id"/>
                        <h2>
                            <field name="brand_id" placeholder="e.g. Tesla"/>
                        </h2>
                    </div>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="vehicle_type"/>
                            <field name="category_id" options="{'no_create_edit': True}"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Information" name="information">
                            <group>
                                <group string="Model" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}">
                                    <field name="seats"/>
                                    <field name="doors"/>
                                    <field name="color"/>
                                    <field name="model_year"/>
                                    <field name="trailer_hook"/>
                                </group>
                                <group id="vehicle_information" string="Vehicle Information" attrs="{'invisible': [('vehicle_type', '!=', 'bike')]}">
                                    <field name="electric_assistance"/>
                                </group>
                            </group>
                            <group string="Engine" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}">
                                <group>
                                    <field name="default_fuel_type" required="1"/>
                                    <label for="default_co2"/>
                                    <div class="o_row" name="default_co2">
                                        <field name="default_co2"/><span>g/km</span>
                                    </div>
                                    <field name="co2_standard"/>
                                    <field name="transmission"/>
                                </group>
                                <group>
                                    <label for="power"/>
                                    <div class="o_row">
                                        <field name="power"/><span>kW</span>
                                    </div>
                                    <field name="horsepower"/>
                                    <field name="horsepower_tax"/>
                                </group>
                            </group>
                        </page>
                        <page string="Vendors" name="vendors">
                            <field name="vendors">
                                <kanban quick_create="false" create="true">
                                    <field name="name"/>
                                    <field name="phone"/>
                                    <field name="email"/>
                                    <templates>
                                        <t t-name="kanban-box">
                                            <div style="position: relative" class="oe_kanban_global_click">
                                                <div>
                                                    <div class="o_kanban_record_title">
                                                        <field name="name"/>
                                                        <div class="o_kanban_details float-end">
                                                            <span class="text-muted">
                                                                <t t-if="record.phone.raw_value"><field name="phone"/><br/></t>
                                                                <t t-if="record.email.raw_value"><field name="email"/></t>
                                                            </span>
                                                        </div>
                                                    </div>
                                                </div>
                                            </div>
                                        </t>
                                    </templates>
                                </kanban>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id='fleet_vehicle_model_view_tree' model='ir.ui.view'>
        <field name="name">fleet.vehicle.model.tree</field>
        <field name="model">fleet.vehicle.model</field>
        <field name="arch" type="xml">
            <tree string="Models">
                <field name="brand_id" />
                <field name="name" />
                <field name="vehicle_count" string="Vehicles"/>
                <field name="category_id" optional="show"/>
                <field name="vehicle_type" optional="show"/>
                <field name="default_co2" optional="hide"/>
            </tree>
        </field>
    </record>

    <record id='fleet_vehicle_model_view_kanban' model='ir.ui.view'>
        <field name="name">fleet.vehicle.model.kanban</field>
        <field name="model">fleet.vehicle.model</field>
        <field name="arch" type="xml">
            <kanban string="Models">
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click oe_kanban_details">
                            <div><strong><field name="name"/></strong></div>
                            <div><field name="brand_id"/></div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id='fleet_vehicle_model_view_search' model='ir.ui.view'>
        <field name="name">fleet.vehicle.model.search</field>
        <field name="model">fleet.vehicle.model</field>
        <field name="arch" type="xml">
            <search string="Vehicles costs" >
                <field name="brand_id" />
                <group expand="1" string="Group By">
                    <filter name="groupby_brand" context="{'group_by' : 'brand_id'}" string="Contains Vehicles"/>
                </group>
            </search>
        </field>
    </record>

    <record id='fleet_vehicle_model_action' model='ir.actions.act_window'>
        <field name="name">Models</field>
        <field name="res_model">fleet.vehicle.model</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{"search_default_groupby_brand" : True,}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new model
          </p><p>
            You can define several models (e.g. A3, A4) for each make (Audi).
          </p>
        </field>
    </record>

    <menuitem name="Fleet" id="menu_root" sequence="220" groups="fleet_group_user" web_icon="fleet,static/description/icon.svg"/>
    <menuitem name="Configuration" parent="menu_root" id="fleet_configuration" sequence="100" groups="fleet_group_manager"/>

    <record id='fleet_vehicle_model_brand_view_tree' model='ir.ui.view'>
        <field name="name">fleet.vehicle.model.brand.tree</field>
        <field name="model">fleet.vehicle.model.brand</field>
        <field name="arch" type="xml">
            <tree string="Model Make">
                <field name="name" />
                <field name="model_count" string="Models"/>
            </tree>
        </field>
    </record>

    <record id='fleet_vehicle_model_brand_view_form' model='ir.ui.view'>
        <field name="name">fleet.vehicle.model.brand.form</field>
        <field name="model">fleet.vehicle.model.brand</field>
        <field name="arch" type="xml">
            <form string="Model Make">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_brand_model" type="object" icon="fa-car" class="oe_stat_button"
                            attrs="{'invisible': [('model_count', '=', 0)]}">
                            <field name="model_count" widget="statinfo" string="Models"/>
                        </button>
                    </div>
                    <group>
                        <div>
                            <field name="image_128" widget="image" class="oe_avatar"/>
                            <h1>
                                <field name="name"/>
                            </h1>
                        </div>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id='fleet_vehicle_model_brand_view_kanban' model='ir.ui.view'>
        <field name="name">fleet.vehicle.model.brandkanban</field>
        <field name="model">fleet.vehicle.model.brand</field>
        <field name="arch" type="xml">
            <kanban default_order="name" action="action_brand_model" type="object">
                <field name="id"/>
                <field name="name" />
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_vignette oe_semantic_html_override oe_kanban_global_click">
                            <div class="o_dropdown_kanban dropdown">
                                <a class="dropdown-toggle o-no-caret btn" role="button" data-bs-toggle="dropdown" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                    <span class="fa fa-ellipsis-v"/>
                                </a>
                                <div class="dropdown-menu" role="menu">
                                    <a role="menuitem" type="open" class="dropdown-item">Configuration</a>
                                </div>
                            </div>
                            <div class="o_kanban_image">
                                <img alt="img" t-att-src="kanban_image('fleet.vehicle.model.brand', 'image_128', record.id.raw_value)" class="o_image_64_max" height="52"/>
                            </div>
                            <div class="oe_kanban_details">
                                <h4 class="oe_partner_heading">
                                    <a type="open" class="o_kanban_record_title">
                                        <field name="name"/>
                                    </a>
                                </h4>
                                <div>
                                    <field name="model_count"/> MODELS
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="fleet_vehicle_model_brand_view_search" model="ir.ui.view">
        <field name="name">fleet.vehicle.model.brand.view.search</field>
        <field name="model">fleet.vehicle.model.brand</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter string="With Models" name="with_models"
                    domain="[('model_count', '>', 0)]"/>
            </search>
        </field>
    </record>

    <record id='fleet_vehicle_model_brand_action' model='ir.actions.act_window'>
        <field name="name">Manufacturers</field>
        <field name="res_model">fleet.vehicle.model.brand</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new manufacturer
          </p>
        </field>
    </record>

    <!-- Model Category -->
    <record id='fleet_vehicle_model_category_action' model='ir.actions.act_window'>
        <field name="name">Categories</field>
        <field name="res_model">fleet.vehicle.model.category</field>
        <field name="view_mode">tree</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new category
            </p>
        </field>
    </record>

    <record id="fleet_vehicle_model_category_view_tree" model="ir.ui.view">
        <field name="name">fleet.vehicle.model.category.view.tree</field>
        <field name="model">fleet.vehicle.model.category</field>
        <field name="arch" type="xml">
            <tree string="Model Category" editable="bottom" default_order="sequence, id">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="fleet_vehicle_model_category_view_form" model="ir.ui.view">
        <field name="name">fleet.vehicle.model.category.view.form</field>
        <field name="model">fleet.vehicle.model.category</field>
        <field name="arch" type="xml">
            <form string="Model Category">
                <sheet>
                    <group>
                        <group>
                            <field name="name"/>
                        </group>
                        <group>
                            <field name="sequence" groups="base.group_no_one"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <menuitem name="Models" parent="fleet_configuration" id="fleet_models_configuration" sequence="10" groups="fleet_group_manager"/>
    <menuitem action="fleet_vehicle_model_brand_action" parent="fleet_models_configuration" id="fleet_vehicle_model_brand_menu" sequence="1"/>
    <menuitem action="fleet_vehicle_model_action" parent="fleet_models_configuration" id="fleet_vehicle_model_menu" sequence="5"/>
    <menuitem action="fleet_vehicle_model_category_action" parent="fleet_models_configuration" id="fleet_vehicle_model_category_menu" sequence="10"/>
</odoo>

```

## File: views\fleet_vehicle_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id='fleet_vehicle_view_form' model='ir.ui.view'>
        <field name="name">fleet.vehicle.form</field>
        <field name="model">fleet.vehicle</field>
        <field name="arch" type="xml">
            <form string="Vehicle" js_class="fleet_form" class="o_fleet_form">
                <field name="service_activity" invisible="1"/>
                <header>
                    <button string="Apply New Driver"
                        class="btn btn-primary"
                        type="object"
                        name="action_accept_driver_change"
                        attrs="{'invisible': [('future_driver_id', '=', False)]}"/>
                    <field name="state_id"  widget="statusbar" options="{'clickable': '1'}"/>
                </header>
                <sheet>
                    <field name="company_id" invisible="1"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="country_code" invisible="1"/>
                    <div class="oe_button_box" name="button_box">
                        <button name="open_assignation_logs"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-history">
                            <field name="history_count" widget="statinfo" string="Drivers History"/>
                        </button>
                        <button name="return_action_to_open"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-book"
                            context="{'xml_id':'fleet_vehicle_log_contract_action', 'search_default_inactive': not active}"
                            help="show the contract for this vehicle">
                            <field name="contract_count" widget="statinfo" string="Contracts"/>
                        </button>
                        <button name="return_action_to_open"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-wrench"
                            context="{'xml_id':'fleet_vehicle_log_services_action', 'search_default_inactive': not active}"
                            attrs="{'invisible': [('service_activity', '!=', 'none')]}"
                            help="show the services logs for this vehicle">
                            <field name="service_count" widget="statinfo" string="Services"/>
                        </button>
                        <button name="return_action_to_open"
                            type="object"
                            class="oe_stat_button text-danger"
                            icon="fa-wrench"
                            context="{'xml_id':'fleet_vehicle_log_services_action', 'search_default_inactive': not active}"
                            attrs="{'invisible': [('service_activity', '!=', 'overdue')]}"
                            help="show the services logs for this vehicle">
                            <field name="service_count" widget="statinfo" string="Services"/>
                        </button>
                        <button name="return_action_to_open"
                            type="object"
                            class="oe_stat_button text-warning"
                            icon="fa-wrench"
                            context="{'xml_id':'fleet_vehicle_log_services_action', 'search_default_inactive': not active}"
                            attrs="{'invisible': [('service_activity', '!=', 'today')]}"
                            help="show the services logs for this vehicle">
                            <field name="service_count" widget="statinfo" string="Services"/>
                        </button>
                        <button name="return_action_to_open"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-tachometer"
                            context="{'xml_id':'fleet_vehicle_odometer_action'}"
                            help="show the odometer logs for this vehicle"
                            attrs="{'invisible': [('vehicle_type', '!=', 'car')]}">
                            <field name="odometer_count" widget="statinfo" string="Odometer"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <field name="image_128" widget='image' class="oe_avatar"/>
                    <div class="oe_title">
                        <label for="model_id"/>
                        <h1>
                            <field name="model_id" placeholder="e.g. Model S"/>
                        </h1>
                        <label for="license_plate"/>
                        <h2>
                            <field name="license_plate" class="oe_inline" placeholder="e.g. PAE 326"/>
                        </h2>
                        <label for="tag_ids" class="me-3"/>
                        <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                    </div>
                    <group col="2">
                        <group string="Driver">
                            <field name="active" invisible="1"/>
                            <field name="vehicle_type" invisible="1"/>
                            <field name="driver_id" domain="['|', ('company_id', '=', False ), ('company_id', '=', company_id)]"/>
                            <field name="future_driver_id"/>
                            <field name="plan_to_change_car" groups="fleet.fleet_group_manager" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}"/>
                            <field name="plan_to_change_bike" groups="fleet.fleet_group_manager"  attrs="{'invisible': [('vehicle_type', '!=', 'bike')]}"/>
                            <field name="next_assignation_date"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <group string="Vehicle">
                            <field name="category_id"/>
                            <field name="acquisition_date" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}"/>
                            <field name="write_off_date" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}"/>
                            <field name="vin_sn"/>
                            <label for="odometer" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}"/>
                            <div class="o_row" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}">
                                <field name="odometer"/>
                                <field name="odometer_unit"/>
                            </div>
                            <field name="manager_id" domain="[('share', '=', False)]"/>
                            <field name="location"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Tax Info">
                            <group>
                                <group string="Fiscality">
                                    <field name="horsepower_tax" widget="monetary"/>
                                </group>
                                <group string="Contract">
                                    <field name="first_contract_date"/>
                                    <field name="car_value" widget="monetary"/>
                                    <field name="net_car_value" widget="monetary"/>
                                    <field name="residual_value" widget="monetary"/>
                                </group>
                            </group>
                        </page>
                        <page string="Model">
                            <group>
                                <group string="Model">
                                    <field name="model_year"/>
                                    <field name="transmission"/>
                                    <field name="color"/>
                                    <field name="seats" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}"/>
                                    <field name="doors" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}"/>
                                    <field name="trailer_hook" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}"/>
                                    <field name="frame_type" attrs="{'invisible': [('vehicle_type', '!=', 'bike')]}"/>
                                    <label for="frame_size" attrs="{'invisible': [('vehicle_type', '!=', 'bike')]}"/>
                                    <div class="o_row" attrs="{'invisible': [('vehicle_type', '!=', 'bike')]}">
                                        <field name="frame_size" /><span>cm</span>
                                    </div>
                                    <field name="electric_assistance" attrs="{'invisible': [('vehicle_type', '!=', 'bike')]}"/>
                                </group>
                                <group string="Engine" attrs="{'invisible': [('vehicle_type', '!=', 'car')]}">
                                    <field name="horsepower"/>
                                    <label for="power"/>
                                    <div class="o_row">
                                        <field name="power"/><span>kW</span>
                                    </div>
                                    <field name="fuel_type"/>
                                    <label for="co2"/>
                                    <div class="o_row" name="co2">
                                        <field name="co2"/><span>g/km</span>
                                    </div>
                                    <field name="co2_standard"/>
                                </group>
                            </group>
                        </page>
                        <page string="Note">
                            <field name="description" nolabel="1" placeholder="Write here any other information related to this vehicle" />
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id='fleet_vehicle_view_tree' model='ir.ui.view'>
        <field name="name">fleet.vehicle.tree</field>
        <field name="model">fleet.vehicle</field>
        <field name="arch" type="xml">
            <tree string="Vehicle" 
                decoration-warning="contract_renewal_due_soon and not contract_renewal_overdue"
                decoration-danger="contract_renewal_overdue"
                multi_edit="1"
                sample="1">
                <field name="active" invisible="1"/>
                <field name="license_plate" readonly="1"/>
                <field name="model_id" readonly="1"/>
                <field name="category_id"/>
                <field name="manager_id" optional="hide"/>
                <field name="driver_id" widget="many2one_avatar" readonly="1" optional="show"/>
                <field name="future_driver_id"  widget="many2one_avatar" readonly="1" optional="show"/>
                <field name="log_drivers" invisible="1"/>
                <field name="vin_sn" readonly="1" optional="hide"/>
                <field name="co2" string="CO2 Emissions g/km" optional="hide"  readonly="1"/>
                <field name="acquisition_date" readonly="1"/>
                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" readonly="1"/>
                <field name="state_id" widget="badge" readonly="1" optional="hide"/>
                <field name="contract_renewal_due_soon" invisible="1"/>
                <field name="contract_renewal_overdue" invisible="1"/>
                <field name="contract_renewal_total" invisible="1"/>
                <field name="contract_state" widget="badge" decoration-info="contract_state == 'open'"
                    decoration-danger="contract_state == 'expired'" optional="hide"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record id="fleet_vehicle_view_search" model="ir.ui.view">
        <field name="name">fleet.vehicle.search</field>
        <field name="model">fleet.vehicle</field>
        <field name="arch" type="xml">
            <search string="All vehicles">
                <field string="Vehicle" name="name" filter_domain="['|', ('name', 'ilike', self), ('license_plate', 'ilike', self)]"/>
                <field string="Drivers" name="log_drivers" filter_domain="[
                    '|', '|',
                    ('log_drivers.driver_id', 'ilike', self),
                    ('driver_id', 'ilike', self),
                    ('future_driver_id', 'ilike', self),
                ]"/>
                <field string="Model" name="model_id"/>
                <field string="License Plate" name="license_plate"/>
                <field name="tag_ids"/>
                <field string="Status" name="state_id"/>
                <field string="Current Driver" name="driver_id"/>
                <filter string="Available" name="available"
                    domain="['&amp;', ('future_driver_id', '=', False), '|', ('driver_id', '=', False), '|', '&amp;', ('plan_to_change_car', '=', True), ('vehicle_type', '=', 'car'), '&amp;', ('plan_to_change_bike', '=', True), ('vehicle_type', '=', 'bike')]"/>
                <filter string="Bikes" name="bikes" domain="[('vehicle_type', '=', 'bike')]"/>
                <filter string="Cars" name="cars" domain="[('vehicle_type', '=', 'car')]"/>
                <filter string="Trailer Hook" name="trailer_hook" domain="[('trailer_hook', '=', True)]"/>
                <filter name="planned" string="Planned for Change" domain="['|', '&amp;', ('vehicle_type', '=', 'bike'), ('plan_to_change_bike', '=', True), '&amp;', ('vehicle_type', '=', 'car'), ('plan_to_change_car', '=', True)]"/>
                <separator/>
                <filter string="Need Action" name="alert_true" domain="['|', ('contract_renewal_due_soon', '=', True), ('contract_renewal_overdue', '=', True)]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <group expand="1" string="Group By">
                    <filter string="Status" name="groupby_status" context="{'group_by': 'state_id'}"/>
                    <filter string="Model" name="groupby_model" context="{'group_by': 'model_id'}"/>
                    <filter string="Brand" name="groupby_make" context="{'group_by': 'brand_id'}"/>
                    <filter string="Fuel Type" name="groupby_fueltype" context="{'group_by': 'fuel_type'}"/>
                </group>
           </search>
        </field>
    </record>


    <record id='fleet_vehicle_view_kanban' model='ir.ui.view'>
        <field name="name">fleet.vehicle.kanban</field>
        <field name="model">fleet.vehicle</field>
        <field name="arch" type="xml">
            <kanban default_group_by="state_id" sample="1">
                <field name="license_plate" />
                <field name="model_id" />
                <field name="driver_id" />
                <field name="future_driver_id" />
                <field name="location" />
                <field name="state_id" />
                <field name="id" />
                <field name="tag_ids" />
                <field name="contract_renewal_due_soon" />
                <field name="contract_renewal_overdue" />
                <field name="contract_renewal_name" />
                <field name="contract_renewal_total" />
                <field name="contract_count" />
                <field name="activity_ids"/>
                <field name="activity_state"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>

                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click o_kanban_record_has_image_fill">
                            <div class="o_kanban_image" t-attf-style="background-image:url('#{kanban_image('fleet.vehicle', 'image_128', record.id.raw_value)}')"/>
                            <div class="oe_kanban_details">
                                <strong class="o_kanban_record_title">
                                    <t t-if="record.license_plate.raw_value"><field name="license_plate"/>:</t> <field name="model_id"/>
                                </strong>
                                <div class="o_kanban_tags_section">
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                </div>
                                <ul>
                                    <li>
                                        <t t-if="record.driver_id.raw_value"><field name="driver_id"/></t>
                                    </li>
                                    <li>
                                        <t t-if="record.future_driver_id.raw_value">Future Driver : <field name="future_driver_id"/></t>
                                    </li>
                                    <li>
                                        <t t-if="record.location.raw_value"><small><i class="fa fa-map-marker" title="Location"></i> <field name="location"/></small></t>
                                    </li>
                                </ul>
                                <div class="o_kanban_record_bottom" t-if="!selection_mode">
                                    <div class="oe_kanban_bottom_left">
                                        <a t-if="record.contract_count.raw_value>0" data-type="object"
                                           data-name="return_action_to_open" href="#" class="oe_kanban_action oe_kanban_action_a"
                                           data-context='{"xml_id":"fleet_vehicle_log_contract_action"}'>
                                            <field name="contract_count"/>
                                            Contract(s)
                                            <span t-if="record.contract_renewal_due_soon.raw_value and !record.contract_renewal_overdue.raw_value"
                                                class="fa fa-exclamation-triangle" t-att-style="'color:orange'" role="img" aria-label="Warning: renewal due soon" title="Warning: renewal due soon">
                                            </span>
                                             <span t-if="record.contract_renewal_overdue.raw_value"
                                                class="fa fa-exclamation-triangle" t-att-style="'color:red;'" role="img" aria-label="Attention: renewal overdue" title="Attention: renewal overdue">
                                            </span>
                                        </a>
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="fleet_vehicle_view_activity" model="ir.ui.view">
        <field name="name">fleet.vehicle.activity</field>
        <field name="model">fleet.vehicle</field>
        <field name="arch" type="xml">
            <activity string="Vehicles">
                <field name="license_plate"/>
                <field name="id"/>
                <templates>
                    <div t-name="activity-box">
                        <img t-att-src="activity_image('fleet.vehicle', 'image_128', record.id.raw_value)" role="img" t-att-title="record.id.value" t-att-alt="record.id.value"/>
                        <div>
                            <field name="license_plate"/> : <field name="model_id"/>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record id="fleet_vehicle_view_pivot" model="ir.ui.view">
       <field name="model">fleet.vehicle</field>
       <field name="arch" type="xml">
            <pivot>
                <field name="state_id" type="col" />
                <field name="brand_id" type="row" />
                <field name="model_id" type="row" />
                <field name="license_plate" type="row" />
            </pivot>
        </field>
    </record>

    <record id='fleet_vehicle_action' model='ir.actions.act_window'>
        <field name="name">Vehicles</field>
        <field name="res_model">fleet.vehicle</field>
        <field name="view_mode">kanban,tree,form,pivot,activity</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Ready to manage your fleet more efficiently ?
          </p><p>
            Let's create your first vehicle.
          </p>
        </field>
    </record>

    <menuitem name="Fleet" parent="menu_root" id="fleet_vehicles" sequence="2" groups="fleet_group_user"/>
    <menuitem action="fleet_vehicle_action" parent="fleet_vehicles" name="Fleet"
        id="fleet_vehicle_menu" groups="fleet_group_user" sequence="0"/>

   <record id='fleet_vehicle_odometer_view_form' model='ir.ui.view'>
        <field name="name">fleet.vehicle.odometer.form</field>
        <field name="model">fleet.vehicle.odometer</field>
        <field name="arch" type="xml">
            <form string="Odometer Logs">
                <sheet>
                    <group>
                        <group>
                            <field name="vehicle_id"/>
                            <label for="value"/>
                            <div class="o_row">
                                <field name="value" class="oe_inline"/>
                                <field name="unit" class="ms-2"/>
                            </div>
                            <field name="date"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id='fleet_vehicle_odometer_view_tree' model='ir.ui.view'>
        <field name="name">fleet.vehicle.odometer.tree</field>
        <field name="model">fleet.vehicle.odometer</field>
        <field name="arch" type="xml">
            <tree string="Odometer Logs" editable="top">
                <field name="date" />
                <field name="vehicle_id"/>
                <field name="driver_id" widget="many2one_avatar"/>
                <field name="value" />
                <field name="unit" />
            </tree>
        </field>
    </record>

    <record id='fleet_vehicle_odometer_view_kanban' model='ir.ui.view'>
        <field name="name">fleet.vehicle.odometer.kanban</field>
        <field name="model">fleet.vehicle.odometer</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div>
                                <strong>
                                    <field name="vehicle_id"/>
                                    <span class="float-end"><field name="date"/></span>
                                </strong>
                            </div>
                            <div>
                                <span><field name="driver_id"/></span>
                                <span class="float-end"><field name="value"/> Km</span>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id='fleet_vehicle_odometer_view_search' model='ir.ui.view'>
        <field name="name">fleet.vehicle.odometer.search</field>
        <field name="model">fleet.vehicle.odometer</field>
        <field name="arch" type="xml">
            <search string="Vehicles odometers" >
                <field name="vehicle_id"/>
                <field name="driver_id"/>
                <field name="value"/>
                <field name="date"/>
                <group expand="0" string="Group By">
                    <filter name="groupby_vehicle" context="{'group_by': 'vehicle_id'}" string="Vehicle"/>
                </group>
            </search>
        </field>
    </record>

    <record id="fleet_vehicle_odometer_view_graph" model="ir.ui.view">
       <field name="name">fleet.vehicle.odometer.graph</field>
       <field name="model">fleet.vehicle.odometer</field>
       <field name="arch" type="xml">
            <graph string="Odometer Values Per Vehicle" sample="1">
                <field name="vehicle_id"/>
                <field name="value" type="measure"/>
            </graph>
        </field>
    </record>

    <record id='fleet_vehicle_odometer_action' model='ir.actions.act_window'>
        <field name="name">Odometers</field>
        <field name="res_model">fleet.vehicle.odometer</field>
        <field name="view_mode">tree,kanban,form,graph</field>
        <field name="context"></field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new odometer log
          </p><p>
            You can add various odometer entries for all vehicles.
          </p>
        </field>
    </record>

    <menuitem action="fleet_vehicle_odometer_action" parent="fleet_vehicles" id="fleet_vehicle_odometer_menu" groups="fleet_group_user" sequence="10"/>

    <record id='fleet_vehicle_service_types_view_tree' model='ir.ui.view'>
        <field name="name">fleet.service.type.tree</field>
        <field name="model">fleet.service.type</field>
        <field name="arch" type="xml">
            <tree string="Service Types" editable="bottom">
                <field name="name" />
                <field name="category"/>
            </tree>
        </field>
    </record>

    <record id="fleet_vehicle_service_types_view_search" model="ir.ui.view">
        <field name="model">fleet.service.type</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="category"/>
                <group expand="1" string="Group By">
                    <filter name="groupby_category" context="{'group_by' : 'category'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id='fleet_vehicle_service_types_action' model='ir.actions.act_window'>
        <field name="name">Types</field>
        <field name="res_model">fleet.service.type</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new type of service
          </p><p>
            Each service can used in contracts, as a standalone service or both.
          </p>
        </field>
    </record>
    <menuitem name="Services" parent="fleet_configuration" id="fleet_services_configuration" sequence="20" groups="base.group_no_one"/>
    <menuitem action="fleet_vehicle_service_types_action" parent="fleet_services_configuration" name="Types"
        id="fleet_vehicle_service_types_menu" sequence="1" groups="base.group_no_one"/>

    <record id='fleet_vehicle_state_view_tree' model='ir.ui.view'>
        <field name="name">fleet.vehicle.state.tree</field>
        <field name="model">fleet.vehicle.state</field>
        <field name="arch" type="xml">
            <tree string="State" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name" />
            </tree>
        </field>
    </record>

    <record id='fleet_vehicle_state_action' model='ir.actions.act_window'>
        <field name="name">Status</field>
        <field name="res_model">fleet.vehicle.state</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new vehicle status
          </p><p>
            You can customize available status to track the evolution of
            each vehicle. Example: active, being repaired, sold.
          </p>
        </field>
    </record>

    <menuitem name="Vehicle" parent="fleet_configuration" id="fleet_vehicles_configuration" sequence="30" groups="base.group_no_one"/>
    <menuitem action="fleet_vehicle_state_action" parent="fleet_vehicles_configuration" id="fleet_vehicle_state_menu" sequence="10" groups="base.group_no_one"/>

    <record id="fleet_vehicle_tag_view_view_form" model="ir.ui.view">
        <field name="name">fleet.vehicle.tag.form</field>
        <field name="model">fleet.vehicle.tag</field>
        <field name="arch" type="xml">
            <form string="Vehicle Tags">
                <sheet>
                    <group>
                        <field name="name"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="fleet_vehicle_tag_view_view_tree" model="ir.ui.view">
        <field name="name">fleet.vehicle.tag.tree</field>
        <field name="model">fleet.vehicle.tag</field>
        <field name="arch" type="xml">
            <tree string="Vehicle Tags" editable="bottom">
                <field name="name"/>
                <field name="color" widget="color_picker"/>
            </tree>
        </field>
    </record>

    <record id="fleet_vehicle_tag_action" model="ir.actions.act_window">
        <field name="name">Tags</field>
        <field name="res_model">fleet.vehicle.tag</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new tag
            </p>
        </field>
    </record>

    <menuitem id="fleet_vehicle_tag_menu" parent="fleet_vehicles_configuration" action="fleet_vehicle_tag_action" sequence="20" groups="base.group_no_one"/>

    <record id="fleet_vehicle_assignation_log_view_list" model="ir.ui.view">
        <field name="name">fleet.vehicle.assignation.log.view.tree</field>
        <field name="model">fleet.vehicle.assignation.log</field>
        <field name="arch" type="xml">
            <tree string="Assignment Logs" editable="bottom">
                <field name="vehicle_id" />
                <field name="driver_id" widget="many2one_avatar" string="Current Driver" />
                <field name="date_start"/>
                <field name="date_end"/>
            </tree>
        </field>
    </record>

</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- Activity types config -->
    <record id="mail_activity_type_action_config_fleet" model="ir.actions.act_window">
        <field name="name">Activity Types</field>
        <field name="res_model">mail.activity.type</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('res_model', '=', False), ('res_model', '=', 'fleet.vehicle.log.contract')]</field>
        <field name="context">{'default_res_model': 'fleet.vehicle.log.contract'}</field>
    </record>
    <menuitem id="fleet_menu_config_activity_type"
        action="mail_activity_type_action_config_fleet"
        parent="fleet_configuration"
        sequence="99"
        groups="base.group_no_one"/>
</odoo>
```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.hr.fleet</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="90"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('settings')]" position="inside">
                    <div class="app_settings_block" data-string="Fleet" id="fleet" string="Fleet" data-key="fleet" groups="fleet.fleet_group_manager">
                        <h2>Fleet Management</h2>
                        <div class="row mt16 o_settings_container" id="end_contract_setting">
                            <div class="col-12 col-lg-6 o_setting_box">
                                <div class="o_setting_left_pane"/>
                                <div class="o_setting_right_pane">
                                    <span class="o_form_label">End Date Contract Alert</span>
                                    <div class="text-muted content-group mt16">
                                        <span>Send an alert </span>
                                        <field name="delay_alert_contract" class="text-center" style="width: 10%; min-width: 4rem;" />
                                        <span> days before the end date</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="fleet_config_settings_action" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'fleet', 'bin_size': False}</field>
        </record>

        <menuitem id="fleet_config_settings_menu" name="Settings"
            parent="fleet.fleet_configuration" sequence="0" action="fleet_config_settings_action"
            groups="base.group_system"/>
    </data>
</odoo>

```

