# Odoo Module: l10n_eg_edi_eta

Category: account

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Egypt E-Invoicing",
    'summary': """
            Egyptian Tax Authority Invoice Integration
        """,
    'description': """
Egypt Tax Authority Invoice Integration
==============================================================================
Integrates with the ETA portal to automatically send and sign the Invoices to the Tax Authority.
    """,
    'author': 'Odoo S.A., Plementus',
    'category': 'account',
    'version': '0.2',
    'license': 'LGPL-3',
    'depends': ['account_edi', 'l10n_eg'],
    'data': [
        'data/account_edi_data.xml',
        'data/l10n_eg_edi.activity.type.csv',
        'data/l10n_eg_edi.uom.code.csv',
        'data/uom.uom.csv',
        'security/ir.model.access.csv',
        'security/eta_thumb_drive_security.xml',
        'views/uom_uom_view.xml',
        'views/account_move_view.xml',
        'views/account_journal_view.xml',
        'views/eta_thumb_drive.xml',
        'views/product_template_views.xml',
        'views/res_config_settings_view.xml',
        'views/report_invoice.xml',
        'data/res_country_data.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'l10n_eg_edi_eta/static/src/js/sign_invoice.js',
        ],
    },
    'external_dependencies': {
        'python': ['asn1crypto'],
    },
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="edi_eg_eta" model="account.edi.format">
            <field name="name">Egyptian Tax Authority</field>
            <field name="code">eg_eta</field>
        </record>
    </data>
</odoo>

```

## File: data\l10n_eg_edi.activity.type.csv

```csv
id,code,name
l10n_eg_activity_type_0111,0111,"Cultivation of grains and crops (except for rice), legumes and oilseeds"
l10n_eg_activity_type_0112,0112,Cultivation of rice
l10n_eg_activity_type_0113,0113,"Growing vegetables, melons, roots and tubers"
l10n_eg_activity_type_0114,0114,Cultivation of sugar cane
l10n_eg_activity_type_0115,0115,Tobacco cultivation
l10n_eg_activity_type_0116,0116,Growing fiber crops
l10n_eg_activity_type_0119,0119,Cultivation of other non-perennial crops
l10n_eg_activity_type_0121,0121,the cultivation of grapevines.
l10n_eg_activity_type_0122,0122,Growing tropical and subtropical fruits
l10n_eg_activity_type_0123,0123,Cultivation of citrus fruits
l10n_eg_activity_type_0124,0124,Cultivation of fruit with Date kernel and from palm trees
l10n_eg_activity_type_0125,0125,Plant fruit trees and shrubs and other nuts
l10n_eg_activity_type_0126,0126,Growing oil fruits
l10n_eg_activity_type_0127,0127,Cultivation of the crops from which drinks are extracted
l10n_eg_activity_type_0128,0128,"Cultivation of spice crops, aromatics, medicine and pharmaceutical drugs"
l10n_eg_activity_type_0129,0129,Cultivation of other perennial crops
l10n_eg_activity_type_0130,0130,Crop breeding
l10n_eg_activity_type_0141,0141,Breeding of cattle and buffalo
l10n_eg_activity_type_0142,0142,Breeding of horses and mare
l10n_eg_activity_type_0143,0143,Breeding of camels
l10n_eg_activity_type_0144,0144,Breeding sheep and goats
l10n_eg_activity_type_0145,0145,Breeding of Pig
l10n_eg_activity_type_0146,0146,Poultry farming
l10n_eg_activity_type_0149,0149,Breeding other animals
l10n_eg_activity_type_0150,0150,Mixed education
l10n_eg_activity_type_0161,0161,Support activities for crop production
l10n_eg_activity_type_0162,0162,Activities in support of animal production
l10n_eg_activity_type_0163,0163,Post-harvest activities
l10n_eg_activity_type_0164,0164,Preparing grains for reproduction
l10n_eg_activity_type_0170,0170,"Hunting, erection, and related service activities"
l10n_eg_activity_type_0210,0210,Forest care and forest-related activities
l10n_eg_activity_type_0220,0220,Wood cutting
l10n_eg_activity_type_0230,0230,Assembling non-wood forest products
l10n_eg_activity_type_0240,0240,Forest support services
l10n_eg_activity_type_0311,0311,Fishing
l10n_eg_activity_type_0312,0312,River fishing
l10n_eg_activity_type_0321,0321,Marine farms
l10n_eg_activity_type_0322,0322,River farms
l10n_eg_activity_type_0411,0411,Earn a job
l10n_eg_activity_type_0412,0412,Income from government agencies salaries
l10n_eg_activity_type_0413,0413,Income and salaries from the public business sector
l10n_eg_activity_type_0414,0414,Income and salaries from the private sector
l10n_eg_activity_type_0415,0415,Income and salary from non-subject entities
l10n_eg_activity_type_0416,0416,Inspection and sting
l10n_eg_activity_type_0441,0441,Income of agricultural lands
l10n_eg_activity_type_0442,0442,Revenue from constructed real estate
l10n_eg_activity_type_0444,0444,Income of real estate activities
l10n_eg_activity_type_0451,0451,Errand stamp
l10n_eg_activity_type_0461,0461,Revenue from non-funders
l10n_eg_activity_type_0462,0462,Revenue of transferred capital
l10n_eg_activity_type_0463,0463,Income earned from abroad
l10n_eg_activity_type_0464,0464,Other categories / miscellaneous other income
l10n_eg_activity_type_0471,0471,Free market revenue
l10n_eg_activity_type_0472,0472,Free Zones revenue
l10n_eg_activity_type_0510,0510,Hard coal mining
l10n_eg_activity_type_0520,0520,Lignite mining
l10n_eg_activity_type_0610,0610,Extract the crude oil
l10n_eg_activity_type_0620,0620,Extract natural gas
l10n_eg_activity_type_0710,0710,Iron ore mining
l10n_eg_activity_type_0721,0721,Uranium and raw thorium mining
l10n_eg_activity_type_0729,0729,Mining other non-ferrous metals
l10n_eg_activity_type_0810,0810,"Quarrying to extract stones, sand and shale"
l10n_eg_activity_type_0891,0891,Chemical minerals and fertilizer extraction
l10n_eg_activity_type_0892,0892,Peat extraction
l10n_eg_activity_type_0893,0893,Salt extraction
l10n_eg_activity_type_0899,0899,Other mining and quarrying activities are not elsewhere classified
l10n_eg_activity_type_0910,0910,Service activities in support of oil and natural gas extraction
l10n_eg_activity_type_0990,0990,Service activities in support of other mining and quarrying activities
l10n_eg_activity_type_1010,1010,Meat processing and preservation
l10n_eg_activity_type_1020,1020,"Manufacture and preservation of fish, crustaceans and mollusks"
l10n_eg_activity_type_1030,1030,Manufacturing and preserving fruits and vegetables
l10n_eg_activity_type_1040,1040,Manufacture of vegetable and animal oils and fats
l10n_eg_activity_type_1050,1050,Dairy products manufacturing
l10n_eg_activity_type_1061,1061,Manufacture of grain mill products
l10n_eg_activity_type_1062,1062,Manufacture of starch and starch products
l10n_eg_activity_type_1071,1071,Manufacturing bakery products
l10n_eg_activity_type_1072,1072,Sugar industry
l10n_eg_activity_type_1073,1073,"Manufacture of cocoa, chocolate and sugar confectionery"
l10n_eg_activity_type_1074,1074,"Manufacturing pasta, strips, couscous and similar starchy products"
l10n_eg_activity_type_1075,1075,Meals and ready-made food industry
l10n_eg_activity_type_1079,1079,Manufacture of other products not classified elsewhere
l10n_eg_activity_type_1080,1080,Prepared animal food industry
l10n_eg_activity_type_1101,1101,"Spirits distilled, refined and mixed"
l10n_eg_activity_type_1102,1102,Winemaking
l10n_eg_activity_type_1103,1103,The manufacture of alcoholic drinks derived from the molten and the manufacture of molten
l10n_eg_activity_type_1104,1104,Manufacturing soft drinks and producing mineral water
l10n_eg_activity_type_1200,1200,Manufacture of tobacco products
l10n_eg_activity_type_1311,1311,Processing and spinning of textile fibers
l10n_eg_activity_type_1312,1312,Textile weave
l10n_eg_activity_type_1313,1313,The textile industry
l10n_eg_activity_type_1391,1391,Manufacture of knitted and crocheted fabrics
l10n_eg_activity_type_1392,1392,"Manufacture of ready-made textile accessories, except garment wear"
l10n_eg_activity_type_1393,1393,Carpet and blanket industry
l10n_eg_activity_type_1394,1394,"Manufacture of ropes, thick and double ropes and nets"
l10n_eg_activity_type_1399,1399,Other textile industry not elsewhere classified
l10n_eg_activity_type_1410,1410,Manufacture of garment with the exception of fur
l10n_eg_activity_type_1420,1420,Fur accessories industry
l10n_eg_activity_type_1430,1430,"The manufacture of clothing, knitted and crocheted"
l10n_eg_activity_type_1511,1511,"Tanning and processing of leather, fillings and dyeing of fur"
l10n_eg_activity_type_1512,1512,"Luggage, handbags and similar industries, along with saddles and horse sets"
l10n_eg_activity_type_1520,1520,Shoe manufacturing
l10n_eg_activity_type_1610,1610,Sawing wood and abrasion
l10n_eg_activity_type_1621,1621,Sheets made of wood veneer and wood-based panels
l10n_eg_activity_type_1622,1622,Manufacture of carpentry accessories intended for buildings and installations
l10n_eg_activity_type_1623,1623,Wooden boxes industry
l10n_eg_activity_type_1629,1629,"Manufacture of wood, wood products and cork, except furniture, and manufacture of articles produced from straw and sheets"
l10n_eg_activity_type_1701,1701,Paper and carvatard pulp industry
l10n_eg_activity_type_1702,1702,Manufacture of corrugated paper and paperboard and boxes made of paper and paperboard
l10n_eg_activity_type_1709,1709,Manufacture of other articles of paper and paperboard
l10n_eg_activity_type_1811,1811,printing
l10n_eg_activity_type_1812,1812,Printing service activities
l10n_eg_activity_type_1820,1820,Clone recorded media
l10n_eg_activity_type_1910,1910,Coke oven products industry
l10n_eg_activity_type_1920,1920,Refined petroleum products
l10n_eg_activity_type_2011,2011,Basic chemicals
l10n_eg_activity_type_2012,2012,Manufacture of fertilizers and nitrogen compounds
l10n_eg_activity_type_2013,2013,Plastics industry in its primary forms and synthetic rubber
l10n_eg_activity_type_2021,2021,Pesticide industry and other agricultural chemical products
l10n_eg_activity_type_2022,2022,"Manufacture of paints, varnishes, and similar coatings, printing inks and molds"
l10n_eg_activity_type_2023,2023,"Manufacture of soap, disinfectants, cleaning and polishing preparations, perfumes and cosmetics"
l10n_eg_activity_type_2029,2029,Manufacture of other chemical products not classified elsewhere
l10n_eg_activity_type_2030,2030,Industrial fiber industry
l10n_eg_activity_type_2100,2100,"Manufacture of pharmaceutical, chemical, and plant products"
l10n_eg_activity_type_2211,2211,"Manufacture of rubber tires and tubes, renewing and rebuilding the outer surfaces of rubber tires"
l10n_eg_activity_type_2219,2219,Manufacture of other rubber products
l10n_eg_activity_type_2220,2220,Plastics industry
l10n_eg_activity_type_2310,2310,Glass and its products industry
l10n_eg_activity_type_2391,2391,Manufacture of fusion products
l10n_eg_activity_type_2392,2392,Manufacture of Shale products for Building
l10n_eg_activity_type_2393,2393,Manufacture of other Porcelain and ceramic products
l10n_eg_activity_type_2394,2394,"Cement, lime and plaster industry"
l10n_eg_activity_type_2395,2395,"Manufacture of concrete products, cement and plaster"
l10n_eg_activity_type_2396,2396,"Cutting, forming and completing the stone processing"
l10n_eg_activity_type_2399,2399,Manufacture of non-metallic minerals products not classified elsewhere
l10n_eg_activity_type_2410,2410,The industry of basic iron and steel
l10n_eg_activity_type_2420,2420,Manufacture of precious and non-ferrous basic metals
l10n_eg_activity_type_2431,2431,Iron and steel casting
l10n_eg_activity_type_2432,2432,Non-ferrous metal casting
l10n_eg_activity_type_2511,2511,Structural metal products industry
l10n_eg_activity_type_2512,2512,Industry of tanks and metal containers
l10n_eg_activity_type_2513,2513,Water vapor generators except for central heating boilers in hot water
l10n_eg_activity_type_2520,2520,Arms and ammunition industry
l10n_eg_activity_type_2591,2591,"Forming metals by hammering, pressing, casting, rolling, and treatment of metal powders"
l10n_eg_activity_type_2592,2592,Metal processing and coating
l10n_eg_activity_type_2593,2593,"Manufacture of cutting tools, hand tools and general metal tools"
l10n_eg_activity_type_2599,2599,Manufacture of other fabricated metal products not classified elsewhere
l10n_eg_activity_type_2610,2610,Electronic components and panels industry
l10n_eg_activity_type_2620,2620,The manufacture of electronic computers and related devices
l10n_eg_activity_type_2630,2630,Communications equipment industry
l10n_eg_activity_type_2640,2640,Electronic devices industry
l10n_eg_activity_type_2651,2651,"Manufacturing measuring, testing, navigation and control devices"
l10n_eg_activity_type_2652,2652,Watch and alarm clock industry
l10n_eg_activity_type_2660,2660,"Radiation, medical and therapeutic electronic devices industry"
l10n_eg_activity_type_2670,2670,Optical equipment and imaging equipment industry
l10n_eg_activity_type_2680,2680,Optical and magnetic conveyor industry
l10n_eg_activity_type_2710,2710,"Manufacture of motors, generators, electrical transformers, devices and control panels for electricity distribution"
l10n_eg_activity_type_2720,2720,Manufacture of dry and stored batteries
l10n_eg_activity_type_2731,2731,Industrial fiber cable industry
l10n_eg_activity_type_2732,2732,Other electrical and electronic wires and cables
l10n_eg_activity_type_2733,2733,Wire devices industry
l10n_eg_activity_type_2740,2740,Electrical lighting devices industry
l10n_eg_activity_type_2750,2750,Home appliances industry
l10n_eg_activity_type_2790,2790,Other electrical appliances industry
l10n_eg_activity_type_2811,2811,"Manufacture of generators and engines, with the exception of aircraft, vehicles and motorcycles"
l10n_eg_activity_type_2812,2812,Liquid power devices industry
l10n_eg_activity_type_2813,2813,"Manufacture of pumps, compressors, tapes and other valves"
l10n_eg_activity_type_2814,2814,"Gears, carriers and driving devices industry"
l10n_eg_activity_type_2815,2815,"Manufacture of furnaces, furnaces and their incinerators"
l10n_eg_activity_type_2816,2816,The elevators and equipment needed for it
l10n_eg_activity_type_2817,2817,Manufacture of office equipment and equipment (excluding electronic computers and their accessories)
l10n_eg_activity_type_2818,2818,Manufacture of manual power steering equipment
l10n_eg_activity_type_2819,2819,Other equipment industry of various purposes
l10n_eg_activity_type_2821,2821,Agricultural and forestry equipment industry
l10n_eg_activity_type_2822,2822,Manufacture of equipment and machinery for forming metals
l10n_eg_activity_type_2823,2823,Metal equipment industry
l10n_eg_activity_type_2824,2824,Mining and quarrying and building equipment industry
l10n_eg_activity_type_2825,2825,"Manufacture of food, beverage and tobacco industries equipment"
l10n_eg_activity_type_2826,2826,"Manufacture of ready-made clothes, accessories, and leather production"
l10n_eg_activity_type_2829,2829,Manufacture of other special-purpose equipment
l10n_eg_activity_type_2910,2910,Manufacture of motor vehicles
l10n_eg_activity_type_2920,2920,Manufacture of motor vehicle bodies and the manufacture of trailers and semi-trailers
l10n_eg_activity_type_2930,2930,Manufacture of accessories and spare parts for motor vehicles
l10n_eg_activity_type_3011,3011,Building ship hulls and rafts
l10n_eg_activity_type_3012,3012,Manufacture of pleasure boats and sport boats
l10n_eg_activity_type_3020,3020,Railroad locomotives and rolling stock industry
l10n_eg_activity_type_3030,3030,Air and spacecraft industry
l10n_eg_activity_type_3040,3040,Manufacture of military military vehicles
l10n_eg_activity_type_3091,3091,Industry
l10n_eg_activity_type_3092,3092,Manufacture of ordinary bicycles and infirm vehicles
l10n_eg_activity_type_3099,3099,Other transportation equipment industry not classified elsewhere
l10n_eg_activity_type_3100,3100,Furniture Industry
l10n_eg_activity_type_3211,3211,Manufacture of jewelry and related items
l10n_eg_activity_type_3212,3212,Manufacture of imitation jewelry and related items
l10n_eg_activity_type_3220,3220,Musical instrument industry
l10n_eg_activity_type_3230,3230,Sports products industry
l10n_eg_activity_type_3240,3240,Make games and play
l10n_eg_activity_type_3250,3250,Manufacturing of dental and medical equipment and tools
l10n_eg_activity_type_3290,3290,Other industries not classified elsewhere
l10n_eg_activity_type_3311,3311,Repair of manufactured metal products
l10n_eg_activity_type_3312,3312,Machinery repair
l10n_eg_activity_type_3313,3313,Repair of electronic and optical devices
l10n_eg_activity_type_3314,3314,Electronic devices repair
l10n_eg_activity_type_3315,3315,"Repair of transport devices, except for motor vehicles"
l10n_eg_activity_type_3319,3319,Repair other devices
l10n_eg_activity_type_3320,3320,Installation of industrial equipment and devices
l10n_eg_activity_type_3510,3510,"Electric generators, transformers and power distributors"
l10n_eg_activity_type_3520,3520,Manufacture of sulfur gas and distribution of gaseous fuels by means of main pipes
l10n_eg_activity_type_3530,3530,Steam supply and air conditioning
l10n_eg_activity_type_3600,3600,"Water collection, treatment and supply"
l10n_eg_activity_type_3700,3700,Sewer
l10n_eg_activity_type_3811,3811,Collection of non-hazardous waste
l10n_eg_activity_type_3812,3812,Collection of hazardous waste
l10n_eg_activity_type_3821,3821,Treatment and disposal of non-hazardous waste
l10n_eg_activity_type_3822,3822,Treatment and disposal of hazardous waste
l10n_eg_activity_type_3830,3830,Material handling
l10n_eg_activity_type_3900,3900,Recycling activities and services and the disposal of other waste
l10n_eg_activity_type_4100,4100,Building constructions
l10n_eg_activity_type_4210,4210,Road and railway constructions
l10n_eg_activity_type_4220,4220,Construction for projects of public benefit
l10n_eg_activity_type_4290,4290,Construction for other civil engineering projects
l10n_eg_activity_type_4311,4311,Remove the installations
l10n_eg_activity_type_4312,4312,Preparing sites
l10n_eg_activity_type_4321,4321,Electrical installations
l10n_eg_activity_type_4322,4322,"Plumbing, heating and air-conditioning installations"
l10n_eg_activity_type_4329,4329,Other structural installations
l10n_eg_activity_type_4330,4330,Completion and finishing of buildings
l10n_eg_activity_type_4390,4390,Other specialized construction activities
l10n_eg_activity_type_4510,4510,Sale of motor vehicles
l10n_eg_activity_type_4520,4520,Maintenance and repair of motor vehicles
l10n_eg_activity_type_4530,4530,Sale of motor vehicle parts and accessories
l10n_eg_activity_type_4540,4540,"Sale, maintenance and repair of motorcycles, parts and accessories thereof"
l10n_eg_activity_type_4610,4610,Wholesale trade on the basis of a contract or a fee
l10n_eg_activity_type_4620,4620,Wholesale trade in agricultural raw materials and live animals
l10n_eg_activity_type_4630,4630,"Wholesale trade of food, beverages and tobacco"
l10n_eg_activity_type_4641,4641,"Wholesale trade of clothes, fabrics and shoes"
l10n_eg_activity_type_4649,4649,Wholesale trade for other household appliances
l10n_eg_activity_type_4651,4651,"Wholesale trade of computer hardware, accessories and computer software"
l10n_eg_activity_type_4652,4652,"Wholesale trade of electronic devices, communications devices and accessories"
l10n_eg_activity_type_4653,4653,"Wholesale trade for agricultural equipment, machinery and supplies"
l10n_eg_activity_type_4659,4659,Wholesale trade of equipment and other devices
l10n_eg_activity_type_4661,4661,"Wholesale trade of dry, liquid and gaseous fuels and related products"
l10n_eg_activity_type_4662,4662,Wholesale trade in precious metals and minerals
l10n_eg_activity_type_4663,4663,"Wholesale trade, supplies and equipment for building materials, hardware, plumbing and heating appliances"
l10n_eg_activity_type_4669,4669,"Wholesale trade for waste, waste and other products not classified elsewhere"
l10n_eg_activity_type_4690,4690,Non-specialized wholesale trade
l10n_eg_activity_type_4711,4711,"Retail sale in non-specialized stores of food, beverages or tobacco"
l10n_eg_activity_type_4719,4719,Other retail types in non-specialized stores
l10n_eg_activity_type_4721,4721,Retail sale in specialized food stores
l10n_eg_activity_type_4722,4722,Retail sale in specialized stores for drinks
l10n_eg_activity_type_4723,4723,Retail sale in specialized stores of tobacco products
l10n_eg_activity_type_4730,4730,Retail sale of specialized vehicles for fuel
l10n_eg_activity_type_4741,4741,"Retail sale in stores specialized in computer hardware, accessories, computer software, and communications equipment"
l10n_eg_activity_type_4751,4751,Retail sale in clothing stores
l10n_eg_activity_type_4752,4752,"Retail sale in specialized stores of hardware, paint and glass"
l10n_eg_activity_type_4753,4753,"Retail sale in specialized stores of carpets, blankets, wall and floor coverings"
l10n_eg_activity_type_4759,4759,"Retail sale in specialized stores of household electrical appliances, furniture, lighting equipment and other household appliances"
l10n_eg_activity_type_4761,4761,"Retail sale in specialized stores of books, newspapers, and stationery"
l10n_eg_activity_type_4762,4762,Retail sale in specialized stores of music and video recordings
l10n_eg_activity_type_4763,4763,Retail sale in specialized stores of sports equipment
l10n_eg_activity_type_4764,4764,Retail sale in specialized games and toys stores
l10n_eg_activity_type_4771,4771,"Retail sale in specialized stores of shoes, clothing and leather products"
l10n_eg_activity_type_4772,4772,"Retail sale in specialized stores of pharmaceutical, medical and pharmaceutical products, ornamental and cosmetic products"
l10n_eg_activity_type_4773,4773,Retail sale in specialized stores of other new products
l10n_eg_activity_type_4774,4774,Retail sale of used products
l10n_eg_activity_type_4781,4781,"Retail sale through kiosks and markets of food, soft drinks and tobacco products"
l10n_eg_activity_type_4782,4782,"Retail sale through kiosks and markets of clothes, fabrics and shoes"
l10n_eg_activity_type_4789,4789,Retail sale via stalls of other products
l10n_eg_activity_type_4742,4742,Retail sale in stores specialized in audio-visual equipment
l10n_eg_activity_type_4791,4791,Retail sale via mail requests or through the Internet
l10n_eg_activity_type_4799,4799,"Other types of retail sales that do not take place in stores, kiosks or markets"
l10n_eg_activity_type_4911,4911,Inland passenger transportation
l10n_eg_activity_type_4912,4912,Shipping by rail
l10n_eg_activity_type_4921,4921,Transporting land passengers outside and inside cities
l10n_eg_activity_type_4922,4922,Other types of passenger transport by land
l10n_eg_activity_type_4923,4923,Land transportation of goods by bus
l10n_eg_activity_type_4930,4930,Pipeline transportation
l10n_eg_activity_type_5011,5011,Transportation of marine and coastal passengers
l10n_eg_activity_type_5012,5012,Marine and coastal cargo transportation
l10n_eg_activity_type_5021,5021,Inland passenger water transport
l10n_eg_activity_type_5022,5022,Inland water transport of goods
l10n_eg_activity_type_5110,5110,Air transport of passengers
l10n_eg_activity_type_5120,5120,Air freight transport
l10n_eg_activity_type_5210,5210,Keep and store
l10n_eg_activity_type_5221,5221,Service activities related to road transport
l10n_eg_activity_type_5222,5222,Emergency service activities related to maritime transport
l10n_eg_activity_type_5223,5223,Emergency service activities related to air transport
l10n_eg_activity_type_5224,5224,Cargo handling
l10n_eg_activity_type_5229,5229,Other activities in support of the transfer
l10n_eg_activity_type_5310,5310,Mail activities
l10n_eg_activity_type_5320,5320,Parcel delivery activities
l10n_eg_activity_type_5510,5510,Short-term placement activities (rental - housing
l10n_eg_activity_type_5520,5520,"Campgrounds, parking lots, and locomotives"
l10n_eg_activity_type_5590,5590,Other types of placement
l10n_eg_activity_type_5610,5610,Restaurant service and food delivery activities by mobile means
l10n_eg_activity_type_5621,5621,Event catering
l10n_eg_activity_type_5629,5629,Other catering services activities
l10n_eg_activity_type_5630,5630,Light beverage service activities
l10n_eg_activity_type_5811,5811,Publishing books
l10n_eg_activity_type_5812,5812,Publish the directory and address lists
l10n_eg_activity_type_5813,5813,"Publishing newspapers, magazines and periodicals"
l10n_eg_activity_type_5819,5819,Other publishing activities
l10n_eg_activity_type_5820,5820,Computer Software Publishing
l10n_eg_activity_type_5911,5911,"Film, video and television program production activities"
l10n_eg_activity_type_5912,5912,"Subsequent activities for the production of movies, videos and television programs"
l10n_eg_activity_type_5913,5913,"Motion picture, video and television program distribution activities"
l10n_eg_activity_type_5914,5914,Film screening activities
l10n_eg_activity_type_5920,5920,Production and publishing of sound and music recordings
l10n_eg_activity_type_6010,6010,Broadcasting over radio stations
l10n_eg_activity_type_6020,6020,Television program preparation and broadcast activities
l10n_eg_activity_type_6110,6110,Wired telecommunications activities
l10n_eg_activity_type_6120,6120,Wireless communication activities
l10n_eg_activity_type_6130,6130,Satellite communication activities
l10n_eg_activity_type_6190,6190,Other telecommunications activities
l10n_eg_activity_type_6201,6201,Computer program preparation activities
l10n_eg_activity_type_6202,6202,Computer consultancy experience and facilities management activities related to computer fields
l10n_eg_activity_type_6209,6209,Other activities related to information technology and computer services
l10n_eg_activity_type_6311,6311,"Data processing, hosting and related activities"
l10n_eg_activity_type_6312,6312,Electronic portals
l10n_eg_activity_type_6391,6391,Activities of press agencies
l10n_eg_activity_type_6399,6399,Other activities for information services that are not classified in other locations
l10n_eg_activity_type_6411,6411,Central banks
l10n_eg_activity_type_6419,6419,Other financial intermediaries
l10n_eg_activity_type_6420,6420,Activities of holding companies
l10n_eg_activity_type_6430,6430,"Credit activities, provision of credits, and similar financial entities"
l10n_eg_activity_type_6491,6491,Financial leasing
l10n_eg_activity_type_6492,6492,Other forms of loans granted
l10n_eg_activity_type_6499,6499,"Other financial services activities, with the exception of insurance and credit provision activities for pensions not classified in other locations"
l10n_eg_activity_type_6511,6511,life insurance
l10n_eg_activity_type_6512,6512,Non-life insurance
l10n_eg_activity_type_6520,6520,re Insurance
l10n_eg_activity_type_6530,6530,Providing credits for pensions
l10n_eg_activity_type_6611,6611,Financial markets management
l10n_eg_activity_type_6612,6612,Security and commodity contracts brokerage
l10n_eg_activity_type_6619,6619,Auxiliary activities for financial services
l10n_eg_activity_type_6621,6621,Risk and damage assessment
l10n_eg_activity_type_6622,6622,Activities of insurance and brokerage agents
l10n_eg_activity_type_6629,6629,Other activities auxiliary to insurance and provision for pensions
l10n_eg_activity_type_6630,6630,Financial credit management activities
l10n_eg_activity_type_6810,6810,Real estate activities with own or leased property
l10n_eg_activity_type_6820,6820,Real estate activities on the basis of a contract or a fee
l10n_eg_activity_type_6910,6910,Legal activities
l10n_eg_activity_type_6920,6920,"Accounting, auditing, bookkeeping and tax advice activities"
l10n_eg_activity_type_7010,7010,The main office activities
l10n_eg_activity_type_7020,7020,Management consultancy activities
l10n_eg_activity_type_7110,7110,Architectural and engineering activities and related technical consulting
l10n_eg_activity_type_7120,7120,Technical tests and analyzes
l10n_eg_activity_type_7210,7210,Research and experimental development in the field of natural and engineering sciences
l10n_eg_activity_type_7220,7220,Experimental research and development in the field of social and human sciences
l10n_eg_activity_type_7310,7310,Advertising
l10n_eg_activity_type_7320,7320,Market studies and public opinion polls
l10n_eg_activity_type_7410,7410,Specialized design activities
l10n_eg_activity_type_7420,7420,Photographic activities
l10n_eg_activity_type_7490,7490,"Other specialized, scientific and artistic activities not classified elsewhere"
l10n_eg_activity_type_7500,7500,Veterinary activities
l10n_eg_activity_type_7710,7710,Renting motor vehicles
l10n_eg_activity_type_7721,7721,Renting and renting sports and leisure products and tools
l10n_eg_activity_type_7722,7722,Rental of video tapes and CDs
l10n_eg_activity_type_7729,7729,Renting and renting other personal and household products
l10n_eg_activity_type_7730,7730,Renting and leasing of other physical devices and equipment
l10n_eg_activity_type_7740,7740,"Rent forms of intellectual property and similar products, except for copyright works"
l10n_eg_activity_type_7810,7810,Activities of recruitment and appointment agencies
l10n_eg_activity_type_7820,7820,Activities of temporary employment agencies
l10n_eg_activity_type_7830,7830,Providing other human resources
l10n_eg_activity_type_7911,7911,Tourism agency services
l10n_eg_activity_type_7912,7912,Activities of tour guides
l10n_eg_activity_type_7990,7990,Other types of reservations and related activities
l10n_eg_activity_type_8010,8010,Private security activities
l10n_eg_activity_type_8020,8020,Security systems services activities
l10n_eg_activity_type_8030,8030,Investigation activities
l10n_eg_activity_type_8110,8110,Support activities for joint facilities
l10n_eg_activity_type_8121,8121,General cleaning of buildings
l10n_eg_activity_type_8129,8129,Building cleaning activities and other industrial facilities
l10n_eg_activity_type_8130,8130,Gardening services and maintenance activities
l10n_eg_activity_type_8211,8211,Joint office support services activities
l10n_eg_activity_type_8219,8219,"Photocopying, document processing and other specialized office support services activities"
l10n_eg_activity_type_8220,8220,Information center services
l10n_eg_activity_type_8230,8230,Organizing trade conferences and exhibitions
l10n_eg_activity_type_8291,8291,Activities of collection agencies and lending offices
l10n_eg_activity_type_8292,8292,Packaging activities
l10n_eg_activity_type_8299,8299,Other support services activities that are not classified in other locations
l10n_eg_activity_type_8411,8411,Public administration activities
l10n_eg_activity_type_8412,8412,"Organizing activities to provide health care, education, educational services and other social services, with the exception of social security"
l10n_eg_activity_type_8413,8413,Organize and contribute to effective business operations
l10n_eg_activity_type_8421,8421,Foreign affairs
l10n_eg_activity_type_8422,8422,Defense activities
l10n_eg_activity_type_8423,8423,Security and public order activities
l10n_eg_activity_type_8430,8430,Compulsory social insurance activities
l10n_eg_activity_type_8510,8510,Primary and pre-primary education
l10n_eg_activity_type_8521,8521,General secondary education
l10n_eg_activity_type_8522,8522,Technical and vocational secondary education
l10n_eg_activity_type_8530,8530,Higher Education
l10n_eg_activity_type_8541,8541,Sports and rehabilitation education
l10n_eg_activity_type_8542,8542,Cultural education
l10n_eg_activity_type_8549,8549,Other types of education not classified elsewhere
l10n_eg_activity_type_8550,8550,Activities in support of education
l10n_eg_activity_type_8610,8610,Hospital activities
l10n_eg_activity_type_8620,8620,Activities related to medicine and dentistry
l10n_eg_activity_type_8690,8690,Other activities related to human health
l10n_eg_activity_type_8710,8710,Nursing facilities for sanatoriums
l10n_eg_activity_type_8720,8720,"Nursing care facilities for special needs clinics, mental illnesses and physical abuse"
l10n_eg_activity_type_8730,8730,Spa facilities for the elderly and disabled
l10n_eg_activity_type_8790,8790,Other spa care facilities
l10n_eg_activity_type_8810,8810,Social work activities for the infirm and disabled that take place without accommodation
l10n_eg_activity_type_8890,8890,Other social business activities that take place without residence
l10n_eg_activity_type_9000,9000,Creative and recreational art activities
l10n_eg_activity_type_9101,9101,Library and archive activities
l10n_eg_activity_type_9102,9102,Museum activities and restoration of historic sites and buildings
l10n_eg_activity_type_9103,9103,Botanical and zoological gardens and natural wildlife activities
l10n_eg_activity_type_9200,9200,Betting activities and gambling
l10n_eg_activity_type_9311,9311,Providing sports facilities
l10n_eg_activity_type_9312,9312,Sports club activities
l10n_eg_activity_type_9319,9319,Other sports activities
l10n_eg_activity_type_9321,9321,Recreational activities and performances in parks
l10n_eg_activity_type_9329,9329,Other leisure and entertainment activities not classified elsewhere
l10n_eg_activity_type_9411,9411,"The activities of commercial enterprises, employers and professional membership organizations"
l10n_eg_activity_type_9412,9412,Activities of professional membership organizations
l10n_eg_activity_type_9420,9420,Trade union activities
l10n_eg_activity_type_9491,9491,Activities of religious organizations
l10n_eg_activity_type_9492,9492,Activities of political organizations
l10n_eg_activity_type_9499,9499,Activities of other membership organizations not classified elsewhere
l10n_eg_activity_type_9511,9511,Computer repair and accessories
l10n_eg_activity_type_9512,9512,Communication equipment repair
l10n_eg_activity_type_9521,9521,Electronic devices repair
l10n_eg_activity_type_9522,9522,"Repair of tools, household appliances, and garden care equipment"
l10n_eg_activity_type_9523,9523,Shoe and leather products repair
l10n_eg_activity_type_9524,9524,Repair of furniture and household items
l10n_eg_activity_type_9529,9529,Repair of other household and personal products
l10n_eg_activity_type_9601,9601,Wash and clean textile and fur products
l10n_eg_activity_type_9602,9602,Hair styling and other cosmetics
l10n_eg_activity_type_9603,9603,Funeral and related activities
l10n_eg_activity_type_9609,9609,Other personal services activities not classified elsewhere
l10n_eg_activity_type_9700,9700,Home employment activities
l10n_eg_activity_type_9810,9810,Activities of producing unearthed products and services for home appliances for personal use
l10n_eg_activity_type_9820,9820,Activities of producing unearthed products and services for home appliances for personal use
l10n_eg_activity_type_9900,9900,Activities of non-regional organizations and bodies

```

## File: data\l10n_eg_edi.uom.code.csv

```csv
id,code,name
l10n_eg_edi_uom_code_A93,A93,Gram/Cubic meter ( g/m3 )
l10n_eg_edi_uom_code_A94,A94,Gram/cubic centimeter ( g/cm3 )
l10n_eg_edi_uom_code_ANN,ANN,Years ( yr )
l10n_eg_edi_uom_code_BAR,BAR,bar ( bar )
l10n_eg_edi_uom_code_BBL,BBL,Barrel (oil 42 gal.)
l10n_eg_edi_uom_code_BG,BG,Bag ( Bag )
l10n_eg_edi_uom_code_BO,BO,Bottle ( Bt. )
l10n_eg_edi_uom_code_BOX,BOX,Box
l10n_eg_edi_uom_code_C45,C45,Nanometer ( nm )
l10n_eg_edi_uom_code_C62,C62,Activity unit ( AU )
l10n_eg_edi_uom_code_CA,CA,Canister ( Can )
l10n_eg_edi_uom_code_CMK,CMK,Square centimeter ( cm2 )
l10n_eg_edi_uom_code_CMQ,CMQ,Cubic centimeter ( cm3 )
l10n_eg_edi_uom_code_CMT,CMT,Centimeter ( cm )
l10n_eg_edi_uom_code_CS,CS,Case ( Case )
l10n_eg_edi_uom_code_CT,CT,Carton ( Car )
l10n_eg_edi_uom_code_CTL,CTL,Centiliter ( Cl )
l10n_eg_edi_uom_code_D10,D10,Siemens per meter ( S/m )
l10n_eg_edi_uom_code_D41,D41,Ton/Cubic meter ( t/m3 )
l10n_eg_edi_uom_code_DAY,DAY,Days ( d )
l10n_eg_edi_uom_code_DMT,DMT,Decimeter ( dm )
l10n_eg_edi_uom_code_EA,EA,each (ST) ( ST )
l10n_eg_edi_uom_code_FAR,FAR,Farad ( F )
l10n_eg_edi_uom_code_FOT,FOT,Foot ( Foot )
l10n_eg_edi_uom_code_FTK,FTK,Square foot ( ft2 )
l10n_eg_edi_uom_code_FTQ,FTQ,Cubic foot ( ft3 )
l10n_eg_edi_uom_code_G42,G42,Microsiemens per centimeter ( microS/cm )
l10n_eg_edi_uom_code_GL,GL,Gram/liter ( g/l )
l10n_eg_edi_uom_code_GLL,GLL,gallon ( gal )
l10n_eg_edi_uom_code_GM,GM,Gram/square meter ( g/m2 )
l10n_eg_edi_uom_code_GPT,GPT,Gallon per thousand
l10n_eg_edi_uom_code_GRM,GRM,Gram ( g )
l10n_eg_edi_uom_code_H63,H63,Milligram/Square centimeter ( mg/cm2 )
l10n_eg_edi_uom_code_HHP,HHP,Hydraulic Horse Power
l10n_eg_edi_uom_code_HLT,HLT,Hectoliter ( hl )
l10n_eg_edi_uom_code_HUR,HUR,Hours ( hrs )
l10n_eg_edi_uom_code_IE,IE,Number of Persons ( PRS )
l10n_eg_edi_uom_code_INH,INH,Inch ( “” )
l10n_eg_edi_uom_code_INK,INK,Square inch ( Inch2 )
l10n_eg_edi_uom_code_IVL,IVL,Interval
l10n_eg_edi_uom_code_JOB,JOB,JOB
l10n_eg_edi_uom_code_KGM,KGM,Kilogram ( KG )
l10n_eg_edi_uom_code_KHZ,KHZ,Kilohertz ( kHz )
l10n_eg_edi_uom_code_KMH,KMH,Kilometer/hour ( km/h )
l10n_eg_edi_uom_code_KMK,KMK,Square kilometer ( km2 )
l10n_eg_edi_uom_code_KMQ,KMQ,Kilogram/cubic meter ( kg/m3 )
l10n_eg_edi_uom_code_KMT,KMT,Kilometer ( km )
l10n_eg_edi_uom_code_KSM,KSM,Kilogram/Square meter ( kg/m2 )
l10n_eg_edi_uom_code_KVT,KVT,Kilovolt ( kV )
l10n_eg_edi_uom_code_KWT,KWT,Kilowatt ( KW )
l10n_eg_edi_uom_code_LB,LB,pounds
l10n_eg_edi_uom_code_LTR,LTR,Liter ( l )
l10n_eg_edi_uom_code_LVL,LVL,Level
l10n_eg_edi_uom_code_M,M,Meter ( m )
l10n_eg_edi_uom_code_MAN,MAN,Man
l10n_eg_edi_uom_code_MGM,MGM,Milligram ( mg )
l10n_eg_edi_uom_code_MIN,MIN,Minute ( min )
l10n_eg_edi_uom_code_MMK,MMK,Square millimeter ( mm2 )
l10n_eg_edi_uom_code_MMQ,MMQ,Cubic millimeter ( mm3 )
l10n_eg_edi_uom_code_MMT,MMT,Millimeter ( mm )
l10n_eg_edi_uom_code_MON,MON,Months ( Months )
l10n_eg_edi_uom_code_MTK,MTK,Square meter ( m2 )
l10n_eg_edi_uom_code_MTQ,MTQ,Cubic meter ( m3 )
l10n_eg_edi_uom_code_ONZ,ONZ,Ounce ( oz )
l10n_eg_edi_uom_code_PAL,PAL,Pascal ( Pa )
l10n_eg_edi_uom_code_PF,PF,Pallet ( PAL )
l10n_eg_edi_uom_code_PK,PK,Pack ( PAK )
l10n_eg_edi_uom_code_PMP,PMP,pump
l10n_eg_edi_uom_code_RUN,RUN,run
l10n_eg_edi_uom_code_SH,SH,Shrink ( Shrink )
l10n_eg_edi_uom_code_SK,SK,Sack
l10n_eg_edi_uom_code_SMI,SMI,Mile ( mile )
l10n_eg_edi_uom_code_ST,ST,"Ton (short,2000 lb)"
l10n_eg_edi_uom_code_TNE,TNE,Tonne ( t )
l10n_eg_edi_uom_code_TON,TON,Ton (metric)
l10n_eg_edi_uom_code_VLT,VLT,Volt ( V )
l10n_eg_edi_uom_code_WEE,WEE,Weeks ( Weeks )
l10n_eg_edi_uom_code_WTT,WTT,Watt ( W )
l10n_eg_edi_uom_code_X03,X03,Meter/Hour ( m/h )
l10n_eg_edi_uom_code_YDQ,YDQ,Cubic yard ( yd3 )
l10n_eg_edi_uom_code_YRD,YRD,Yards ( yd )

```

## File: data\neutralize.sql

```sql
-- disable l10n_eg_edi_eta integration
UPDATE res_company
   SET l10n_eg_production_env = false,
       l10n_eg_client_secret = 'dummy';

```

## File: data\res_country_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="eg_partner_address_form" model="ir.ui.view">
        <field name="name">eg.partner.form.address</field>
        <field name="model">res.partner</field>
        <field name="priority" eval="900"/>
        <field name="arch" type="xml">
            <form>
                <div class="o_address_format">
                    <field name="parent_id" invisible="1"/>
                    <field name="type" invisible="1"/>
                    <field name="l10n_eg_building_no" placeholder="Building Number..." class="o_address_street"/>
                    <field name="street" placeholder="Street" class="o_address_street"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="street2" invisible="1"/>
                    <field name="city"/>
                    <field name="state_id" class="o_address_state" placeholder="State..." options='{"no_open": True}'
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="zip" placeholder="ZIP" class="o_address_zip"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                </div>
            </form>
        </field>
    </record>
    <record id="base.eg" model="res.country">
        <field name="address_view_id" ref="eg_partner_address_form" />
        <field name="address_format" eval="'%(l10n_eg_building_no)s %(street)s\n%(city)s %(state_name)s\n%(zip)s\n%(country_name)s'"/>
    </record>
</odoo>

```

## File: data\uom.uom.csv

```csv
id,l10n_eg_unit_code_id/id
uom.product_uom_day,l10n_eg_edi_uom_code_DAY
uom.product_uom_hour,l10n_eg_edi_uom_code_HUR
uom.product_uom_litre,l10n_eg_edi_uom_code_LTR
uom.product_uom_unit,l10n_eg_edi_uom_code_C62
uom.product_uom_cm,l10n_eg_edi_uom_code_CMT
uom.product_uom_floz,l10n_eg_edi_uom_code_ONZ
uom.product_uom_foot,l10n_eg_edi_uom_code_FOT
uom.product_uom_cubic_foot,l10n_eg_edi_uom_code_FTQ
uom.product_uom_gram,l10n_eg_edi_uom_code_GRM
uom.product_uom_gal,l10n_eg_edi_uom_code_GLL
uom.product_uom_inch,l10n_eg_edi_uom_code_INH
uom.product_uom_kgm,l10n_eg_edi_uom_code_KGM
uom.product_uom_km,l10n_eg_edi_uom_code_KMT
uom.product_uom_lb,l10n_eg_edi_uom_code_LB
uom.product_uom_meter,l10n_eg_edi_uom_code_M
uom.product_uom_mile,l10n_eg_edi_uom_code_SMI
uom.product_uom_cubic_meter,l10n_eg_edi_uom_code_MTQ
uom.product_uom_oz,l10n_eg_edi_uom_code_ONZ
```

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


import json
import logging
import requests
from werkzeug.urls import url_quote
from base64 import b64encode
from odoo.addons.account.tools import LegacyHTTPAdapter
from json.decoder import JSONDecodeError

from odoo import api, models, _
from odoo.tools.float_utils import json_float_round

_logger = logging.getLogger(__name__)


ETA_DOMAINS = {
    'preproduction': 'https://api.preprod.invoicing.eta.gov.eg',
    'production': 'https://api.invoicing.eta.gov.eg',
    'invoice.preproduction': 'https://preprod.invoicing.eta.gov.eg/',
    'invoice.production': 'https://invoicing.eta.gov.eg',
    'token.preproduction': 'https://id.preprod.eta.gov.eg',
    'token.production': 'https://id.eta.gov.eg',
}


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    @api.model
    def _l10n_eg_get_eta_qr_domain(self, production_enviroment=False):
        return production_enviroment and ETA_DOMAINS['invoice.production'] or ETA_DOMAINS['invoice.preproduction']

    @api.model
    def _l10n_eg_get_eta_api_domain(self, production_enviroment=False):
        return production_enviroment and ETA_DOMAINS['production'] or ETA_DOMAINS['preproduction']

    @api.model
    def _l10n_eg_get_eta_token_domain(self, production_enviroment=False):
        return production_enviroment and ETA_DOMAINS['token.production'] or ETA_DOMAINS['token.preproduction']

    @api.model
    def _l10n_eg_eta_connect_to_server(self, request_data, request_url, method, is_access_token_req=False, production_enviroment=False):
        api_domain = is_access_token_req and self._l10n_eg_get_eta_token_domain(production_enviroment) or self._l10n_eg_get_eta_api_domain(production_enviroment)
        request_url = api_domain + request_url
        try:
            session = requests.session()
            session.mount("https://", LegacyHTTPAdapter())
            request_response = session.request(method, request_url, data=request_data.get('body'), headers=request_data.get('header'), timeout=(5, 10))
        except (ValueError, requests.exceptions.ConnectionError, requests.exceptions.MissingSchema, requests.exceptions.Timeout, requests.exceptions.HTTPError) as ex:
            return {
                'error': str(ex),
                'blocking_level': 'warning'
            }
        if not request_response.ok:
            try:
                response_data = request_response.json()
            except JSONDecodeError as ex:
                return {
                    'error': str(ex),
                    'blocking_level': 'error'
                }
            if response_data and response_data.get('error'):
                return {
                    'error': response_data.get('error'),
                    'blocking_level': 'error'
                }
        return {'response': request_response}

    @api.model
    def _l10n_eg_edi_round(self, amount, precision_digits=5):
        """
            This method is call for rounding.
            If anything is wrong with rounding then we quick fix in method
        """
        return json_float_round(amount, precision_digits)

    @api.model
    def _l10n_eg_edi_post_invoice_web_service(self, invoice):
        access_data = self._l10n_eg_eta_get_access_token(invoice)
        if access_data.get('error'):
            return access_data
        invoice_json = json.loads(invoice.l10n_eg_eta_json_doc_id.raw)
        request_url = '/api/v1.0/documentsubmissions'
        request_data = {
            'body': json.dumps({'documents': [invoice_json['request']]}, ensure_ascii=False, indent=4).encode('utf-8'),
            'header': {'Content-Type': 'application/json', 'Authorization': 'Bearer %s' % access_data.get('access_token')}
        }
        response_data = self._l10n_eg_eta_connect_to_server(request_data, request_url, 'POST', production_enviroment=invoice.company_id.l10n_eg_production_env)
        if response_data.get('error'):
            return response_data
        response_data = response_data.get('response').json()
        if response_data.get('rejectedDocuments', False) and isinstance(response_data.get('rejectedDocuments'), list):
            return {
                'error': str(response_data.get('rejectedDocuments')[0].get('error')),
                'blocking_level': 'error'
            }
        if response_data.get('submissionId') is not None and response_data.get('acceptedDocuments'):
            invoice_json['response'] = {
                'l10n_eg_uuid': response_data['acceptedDocuments'][0].get('uuid'),
                'l10n_eg_long_id': response_data['acceptedDocuments'][0].get('longId'),
                'l10n_eg_internal_id': response_data['acceptedDocuments'][0].get('internalId'),
                'l10n_eg_hash_key': response_data['acceptedDocuments'][0].get('hashKey'),
                'l10n_eg_submission_number': response_data['submissionId'],
            }
            invoice.l10n_eg_eta_json_doc_id.raw = json.dumps(invoice_json)
            return {'attachment': invoice.l10n_eg_eta_json_doc_id}
        return {
            'error': _('an Unknown error has occurred'),
            'blocking_level': 'warning'
        }

    @api.model
    def _cancel_invoice_edi_eta(self, invoice):
        access_data = self._l10n_eg_eta_get_access_token(invoice)
        if access_data.get('error'):
            return access_data
        # Check current status. It may already be cancelled or rejected.
        if invoice.l10n_eg_submission_number:
            document_summary = self._l10n_eg_get_einvoice_document_summary(invoice)
            if document_summary.get('doc_data') and document_summary['doc_data'][0].get('status') in ('Cancelled', 'Rejected'):
                return {'success': True}
        request_url = f'/api/v1/documents/state/{url_quote(invoice.l10n_eg_uuid)}/state'
        request_data = {
            'body': json.dumps({'status': 'cancelled', 'reason': 'Cancelled'}),
            'header': {'Content-Type': 'application/json', 'Authorization': 'Bearer %s' % access_data.get('access_token')}
        }
        response_data = self._l10n_eg_eta_connect_to_server(request_data, request_url, 'PUT', production_enviroment=invoice.company_id.l10n_eg_production_env)
        if response_data.get('error'):
            return response_data
        if response_data.get('response').ok:
            return {'success': True}
        return {
            'error': _('an Unknown error has occurred'),
            'blocking_level': 'warning'
        }

    @api.model
    def _l10n_eg_get_einvoice_document_summary(self, invoice):
        access_data = self._l10n_eg_eta_get_access_token(invoice)
        if access_data.get('error'):
            return access_data
        request_url = f'/api/v1.0/documentsubmissions/{url_quote(invoice.l10n_eg_submission_number)}'
        request_data = {
            'body': None,
            'header': {'Content-Type': 'application/json', 'Authorization': 'Bearer %s' % access_data.get('access_token')}
        }
        response_data = self._l10n_eg_eta_connect_to_server(request_data, request_url, 'GET', production_enviroment=invoice.company_id.l10n_eg_production_env)
        if response_data.get('error'):
            return response_data
        response_data = response_data.get('response').json()
        document_summary = [doc for doc in response_data.get('documentSummary', []) if doc.get('uuid') == invoice.l10n_eg_uuid]
        return {'doc_data': document_summary}

    @api.model
    def _l10n_eg_get_einvoice_status(self, invoice):
        document_summary = self._l10n_eg_get_einvoice_document_summary(invoice)
        return_dict = {
            'Invalid': {
                'error': _("This invoice has been marked as invalid by the ETA. Please check the ETA website for more information"),
                'blocking_level': 'error'
            },
            'Submitted': {
                'error': _("This invoice has been sent to the ETA, but we are still awaiting validation"),
                'blocking_level': 'info'
            },
            'Valid': {'success': True},
            'Cancelled': {'error': _('Document Canceled'), 'blocking_level': 'error'},
        }
        if document_summary.get('doc_data') and return_dict.get(document_summary['doc_data'][0].get('status')):
            return return_dict.get(document_summary['doc_data'][0]['status'])
        return {'error': _('an Unknown error has occured'), 'blocking_level': 'warning'}

    def _l10n_eg_eta_get_access_token(self, invoice):
        user = invoice.company_id.sudo().l10n_eg_client_identifier
        secret = invoice.company_id.sudo().l10n_eg_client_secret
        access = '%s:%s' % (user, secret)
        user_and_pass = b64encode(access.encode()).decode()
        request_url = '/connect/token'
        request_data = {'body': {'grant_type': 'client_credentials'}, 'header': {'Authorization': f'Basic {user_and_pass}'}}
        response_data = self._l10n_eg_eta_connect_to_server(request_data, request_url, 'POST', is_access_token_req=True, production_enviroment=invoice.company_id.l10n_eg_production_env)
        if response_data.get('error'):
            return response_data
        return {'access_token' : response_data.get('response').json().get('access_token')}

    @api.model
    def _l10n_eg_get_eta_invoice_pdf(self, invoice):
        access_data = self._l10n_eg_eta_get_access_token(invoice)
        if access_data.get('error'):
            return access_data
        request_url = f'/api/v1.0/documents/{url_quote(invoice.l10n_eg_uuid)}/pdf'
        request_data = {'body': None, 'header': {'Content-Type': 'application/json', 'Authorization': 'Bearer %s' % access_data.get('access_token')}}
        response_data = self._l10n_eg_eta_connect_to_server(request_data, request_url, 'GET', production_enviroment=invoice.company_id.l10n_eg_production_env)
        if response_data.get('error'):
            return response_data
        response_data = response_data.get('response')
        _logger.warning('PDF Function Response %s.', response_data)
        if response_data.ok:
            return {'data': response_data.content}
        else:
            return {'error': _('PDF Document is not available')}

    @api.model
    def _l10n_eg_validate_info_address(self, partner_id, issuer=False, invoice=False):
        fields = ["country_id",
                  "state_id", "city", "street",
                  "l10n_eg_building_no"]
        if (invoice and invoice.amount_total >= invoice.company_id.l10n_eg_invoicing_threshold) or self._l10n_eg_get_partner_tax_type(partner_id, issuer) != 'P':
            fields.append('vat')
        return all(partner_id[field] for field in fields)

    @api.model
    def _l10n_eg_eta_prepare_eta_invoice(self, invoice):

        def group_tax_retention(base_line, tax_values):
            tax = tax_values['tax_repartition_line'].tax_id
            return {'l10n_eg_eta_code': tax.l10n_eg_eta_code.split('_')[0]}

        date_string = invoice.invoice_date.strftime('%Y-%m-%dT%H:%M:%SZ')
        grouped_taxes = invoice._prepare_edi_tax_details(grouping_key_generator=group_tax_retention)
        invoice_line_data, totals = self._l10n_eg_eta_prepare_invoice_lines_data(invoice, grouped_taxes['tax_details_per_record'])
        eta_invoice = {
            'issuer': self._l10n_eg_eta_prepare_address_data(invoice.journal_id.l10n_eg_branch_id, invoice, issuer=True,),
            'receiver': self._l10n_eg_eta_prepare_address_data(invoice.partner_id, invoice),
            'documentType': 'i' if invoice.move_type == 'out_invoice' else 'c' if invoice.move_type == 'out_refund' else 'd' if invoice.move_type == 'in_refund' else '',
            'documentTypeVersion': '1.0',
            'dateTimeIssued': date_string,
            'taxpayerActivityCode': invoice.journal_id.l10n_eg_activity_type_id.code,
            'internalID': invoice.name,
        }
        eta_invoice.update({
            'invoiceLines': invoice_line_data,
            'taxTotals': [{
                'taxType': tax['l10n_eg_eta_code'].split('_')[0].upper(),
                'amount': self._l10n_eg_edi_round(abs(tax['tax_amount'])),
            } for tax in grouped_taxes['tax_details'].values()],
            'totalDiscountAmount': self._l10n_eg_edi_round(totals['discount_total']),
            'totalSalesAmount': self._l10n_eg_edi_round(totals['total_price_subtotal_before_discount']),
            'netAmount': self._l10n_eg_edi_round(abs(invoice.amount_untaxed_signed)),
            'totalAmount': self._l10n_eg_edi_round(abs(invoice.amount_total_signed)),
            'extraDiscountAmount': 0.0,
            'totalItemsDiscountAmount': 0.0,
        })
        if invoice.ref:
            eta_invoice['purchaseOrderReference'] = invoice.ref
        if invoice.invoice_origin:
            eta_invoice['salesOrderReference'] = invoice.invoice_origin
        return eta_invoice

    @api.model
    def _l10n_eg_eta_prepare_invoice_lines_data(self, invoice, tax_data):
        lines = []
        totals = {
            'discount_total': 0.0,
            'total_price_subtotal_before_discount' : 0.0,
        }
        for line in invoice.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section')):
            line_tax_details = tax_data.get(line, {})
            price_unit = self._l10n_eg_edi_round(abs((line.balance / line.quantity) / (1 - (line.discount / 100.0)))) if line.quantity and line.discount != 100.0 else line.price_unit
            price_subtotal_before_discount = self._l10n_eg_edi_round(abs(line.balance / (1 - (line.discount / 100)))) if line.discount != 100.0 else self._l10n_eg_edi_round(price_unit * line.quantity)
            discount_amount = self._l10n_eg_edi_round(price_subtotal_before_discount - abs(line.balance))
            item_code = line.product_id.l10n_eg_eta_code or line.product_id.barcode
            lines.append({
                'description': line.name,
                'itemType': item_code.startswith('EG') and 'EGS' or 'GS1',
                'itemCode': item_code,
                'unitType': line.product_uom_id.l10n_eg_unit_code_id.code,
                'quantity': line.quantity,
                'internalCode': line.product_id.default_code or '',
                'valueDifference': 0.0,
                'totalTaxableFees': 0.0,
                'itemsDiscount': 0.0,
                'unitValue': {
                    'currencySold': invoice.currency_id.name,
                    'amountEGP': price_unit,
                },
                'discount': {
                    'rate': line.discount,
                    'amount': discount_amount,
                },
                'taxableItems': [
                    {
                        'taxType': tax['tax_repartition_line'].tax_id.l10n_eg_eta_code.split('_')[0].upper().upper(),
                        'amount': self._l10n_eg_edi_round(abs(tax['tax_amount'])),
                        'subType': tax['tax_repartition_line'].tax_id.l10n_eg_eta_code.split('_')[1].upper(),
                        **({'rate': abs(tax['tax_repartition_line'].tax_id.amount)} if tax['tax_repartition_line'].tax_id.amount_type != 'fixed' else {}),
                    }
                for tax_details in line_tax_details.get('tax_details', {}).values() for tax in tax_details.get('group_tax_details')
                ],
                'salesTotal': price_subtotal_before_discount,
                'netTotal': self._l10n_eg_edi_round(abs(line.balance)),
                'total': self._l10n_eg_edi_round(abs(line.balance) + line_tax_details.get('tax_amount', 0.0)),
            })
            totals['discount_total'] += discount_amount
            totals['total_price_subtotal_before_discount'] += price_subtotal_before_discount
            if invoice.currency_id != self.env.ref('base.EGP'):
                lines[-1]['unitValue']['currencyExchangeRate'] = self._l10n_eg_edi_round(invoice._l10n_eg_edi_exchange_currency_rate())
                lines[-1]['unitValue']['amountSold'] = line.price_unit
        return lines, totals

    @api.model
    def _l10n_eg_get_partner_tax_type(self, partner_id, issuer=False):
        if issuer:
            return 'B'
        elif partner_id.commercial_partner_id.country_code == 'EG':
            return 'B' if partner_id.commercial_partner_id.is_company else 'P'
        else:
            return 'F'

    @api.model
    def _l10n_eg_eta_prepare_address_data(self, partner, invoice, issuer=False):
        address = {
            'address': {
                'country': partner.country_id.code,
                'governate': partner.state_id.name or '',
                'regionCity': partner.city or '',
                'street': partner.street or '',
                'buildingNumber': partner.l10n_eg_building_no or '',
                'postalCode': partner.zip or '',
            },
            'name': partner.name,
        }
        if issuer:
            address['address']['branchID'] = invoice.journal_id.l10n_eg_branch_identifier or ''
        individual_type = self._l10n_eg_get_partner_tax_type(partner, issuer)
        address['type'] = individual_type or ''
        if invoice.amount_total >= invoice.company_id.l10n_eg_invoicing_threshold or individual_type != 'P':
            address['id'] = partner.vat or ''
        return address

    # -------------------------------------------------------------------------
    # EDI OVERRIDDEN METHODS
    # -------------------------------------------------------------------------

    def _needs_web_services(self):
        return self.code == 'eg_eta' or super()._needs_web_services()

    def _get_move_applicability(self, move):
        # EXTENDS account_edi
        self.ensure_one()
        if self.code != 'eg_eta':
            return super()._get_move_applicability(move)

        if move.is_invoice(include_receipts=True) and move.country_code == 'EG':
            return {
                'post': self._l10n_eg_edi_post_invoice,
                'cancel': self._l10n_eg_edi_cancel_invoice,
                'edi_content': self._l10n_eg_edi_xml_invoice_content,
            }

    def _check_move_configuration(self, invoice):
        errors = super()._check_move_configuration(invoice)
        if self.code != 'eg_eta':
            return errors

        if invoice.journal_id.l10n_eg_branch_id.vat == invoice.partner_id.vat:
            errors.append(_("You cannot issue an invoice to a partner with the same VAT number as the branch."))
        if not self._l10n_eg_get_eta_token_domain(invoice.company_id.l10n_eg_production_env):
            errors.append(_("Please configure the token domain from the system parameters"))
        if not self._l10n_eg_get_eta_api_domain(invoice.company_id.l10n_eg_production_env):
            errors.append(_("Please configure the API domain from the system parameters"))
        if not all([invoice.journal_id.l10n_eg_branch_id, invoice.journal_id.l10n_eg_branch_identifier, invoice.journal_id.l10n_eg_activity_type_id]):
            errors.append(_("Please set the all the ETA information on the invoice's journal"))
        if not self._l10n_eg_validate_info_address(invoice.journal_id.l10n_eg_branch_id):
            errors.append(_("Please add all the required fields in the branch details"))
        if not self._l10n_eg_validate_info_address(invoice.partner_id, invoice=invoice):
            errors.append(_("Please add all the required fields in the customer details"))
        if not all(aml.product_uom_id.l10n_eg_unit_code_id.code for aml in invoice.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section'))):
            errors.append(_("Please make sure the invoice lines UoM codes are all set up correctly"))
        if not all(tax.l10n_eg_eta_code for tax in invoice.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section')).tax_ids):
            errors.append(_("Please make sure the invoice lines taxes all have the correct ETA tax code"))
        if not all(aml.product_id.l10n_eg_eta_code or aml.product_id.barcode for aml in invoice.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section'))):
            errors.append(_("Please make sure the EGS/GS1 Barcode is set correctly on all products"))
        return errors

    def _l10n_eg_edi_post_invoice(self, invoice):
        # In case we have already sent it, but have not got a final answer yet.
        if invoice.l10n_eg_submission_number:
            return {invoice: self._l10n_eg_get_einvoice_status(invoice)}

        if not invoice.l10n_eg_eta_json_doc_id:
            return {
                invoice: {
                    'error':  _("An error occured in created the ETA invoice, please retry signing"),
                    'blocking_level': 'error'
                }
            }
        invoice_json = json.loads(invoice.l10n_eg_eta_json_doc_id.raw)['request']
        if not invoice_json.get('signatures'):
            return {
                invoice: {
                    'error':  _("Please make sure the invoice is signed"),
                    'blocking_level': 'error'
                }
            }
        return {invoice: self._l10n_eg_edi_post_invoice_web_service(invoice)}

    def _l10n_eg_edi_cancel_invoice(self, invoice):
        return {invoice: self._cancel_invoice_edi_eta(invoice)}

    def _l10n_eg_edi_xml_invoice_content(self, invoice):
        return json.dumps(self._l10n_eg_eta_prepare_eta_invoice(invoice)).encode()

    def _is_compatible_with_journal(self, journal):
        # OVERRIDE
        if self.code != 'eg_eta':
            return super()._is_compatible_with_journal(journal)
        return journal.country_code == 'EG' and journal.type == 'sale'

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models, fields


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    l10n_eg_branch_id = fields.Many2one('res.partner', string='Branch', copy=False,
                                        help="Address of the subdivision of the company.  You can just put the "
                                             "company partner if this is used for the main branch.")
    l10n_eg_activity_type_id = fields.Many2one('l10n_eg_edi.activity.type', 'ETA Activity Code', copy=False,
                                               help='This is the activity type of the branch according to Egyptian Tax Authority')
    l10n_eg_branch_identifier = fields.Char('ETA Branch ID', copy=False,
                                            help="This number can be found on the taxpayer profile on the eInvoicing portal. ")

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import json

from odoo import api, models, fields, _
from odoo.exceptions import ValidationError, UserError
from odoo.tools import float_is_zero
from odoo.tools.sql import column_exists, create_column
from datetime import datetime

_logger = logging.getLogger(__name__)


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_eg_long_id = fields.Char(string='ETA Long ID', compute='_compute_eta_long_id')
    l10n_eg_qr_code = fields.Char(string='ETA QR Code', compute='_compute_eta_qr_code_str')
    l10n_eg_submission_number = fields.Char(string='Submission ID', compute='_compute_eta_response_data', store=True, copy=False)
    l10n_eg_uuid = fields.Char(string='Document UUID', compute='_compute_eta_response_data', store=True, copy=False)
    l10n_eg_eta_json_doc_id = fields.Many2one('ir.attachment', copy=False)
    l10n_eg_signing_time = fields.Datetime('Signing Time', copy=False)
    l10n_eg_is_signed = fields.Boolean(copy=False)

    def _auto_init(self):
        if not column_exists(self.env.cr, "account_move", "l10n_eg_uuid"):
            create_column(self.env.cr, "account_move", "l10n_eg_uuid", "VARCHAR")
            # Since l10n_eg_uuid columns does not exist we can assume l10n_eg_submission_number doesn't exist either
            create_column(self.env.cr, "account_move", "l10n_eg_submission_number", "VARCHAR")
        return super()._auto_init()

    @api.depends('l10n_eg_eta_json_doc_id.raw')
    def _compute_eta_long_id(self):
        for rec in self:
            response_data = rec.l10n_eg_eta_json_doc_id and json.loads(rec.l10n_eg_eta_json_doc_id.raw).get('response')
            if response_data:
                rec.l10n_eg_long_id = response_data.get('l10n_eg_long_id')
            else:
                rec.l10n_eg_long_id = False

    @api.depends('invoice_date', 'l10n_eg_uuid', 'l10n_eg_long_id')
    def _compute_eta_qr_code_str(self):
        for move in self:
            if move.invoice_date and move.l10n_eg_uuid and move.l10n_eg_long_id:
                is_prod = move.company_id.l10n_eg_production_env
                base_url = self.env['account.edi.format']._l10n_eg_get_eta_qr_domain(production_enviroment=is_prod)
                qr_code_str = '%s/documents/%s/share/%s' % (base_url, move.l10n_eg_uuid, move.l10n_eg_long_id)
                move.l10n_eg_qr_code = qr_code_str
            else:
                move.l10n_eg_qr_code = ''

    @api.depends('l10n_eg_eta_json_doc_id.raw')
    def _compute_eta_response_data(self):
        for rec in self:
            response_data = rec.l10n_eg_eta_json_doc_id and json.loads(rec.l10n_eg_eta_json_doc_id.raw).get('response')
            if response_data:
                rec.l10n_eg_uuid = response_data.get('l10n_eg_uuid')
                rec.l10n_eg_submission_number = response_data.get('l10n_eg_submission_number')
                rec.l10n_eg_long_id = response_data.get('l10n_eg_long_id')
            else:
                rec.l10n_eg_uuid = False
                rec.l10n_eg_submission_number = False
                rec.l10n_eg_long_id = False

    def button_draft(self):
        self.l10n_eg_eta_json_doc_id = False
        self.l10n_eg_is_signed = False
        return super().button_draft()

    def action_post_sign_invoices(self):
        # only sign invoices that are confirmed and not yet sent to the ETA.
        invoices = self.filtered(lambda r: r.country_code == 'EG' and r.state == 'posted' and not r.l10n_eg_submission_number and r.edi_document_ids.filtered(lambda e: e.edi_format_id.code == 'eg_eta'))
        if not invoices:
            return

        company_ids = invoices.mapped('company_id')
        # since the middleware accepts only one drive at a time, we have to limit signing to one company at a time
        if len(company_ids) > 1:
            raise UserError(_('Please only sign invoices from one company at a time'))

        company_id = company_ids[0]
        drive_id = self.env['l10n_eg_edi.thumb.drive'].search([('user_id', '=', self.env.user.id),
                                                               ('company_id', '=', company_id.id)])

        if not drive_id:
            raise ValidationError(_('Please setup a personal drive for company %s', company_id.name))

        if not drive_id.certificate:
            raise ValidationError(_('Please setup the certificate on the thumb drive menu'))

        invoices.write({'l10n_eg_signing_time': datetime.utcnow()})

        for invoice in invoices:
            eta_invoice = self.env['account.edi.format']._l10n_eg_eta_prepare_eta_invoice(invoice)
            attachment = self.env['ir.attachment'].create({
                    'name': _('ETA_INVOICE_DOC_%s', invoice.name),
                    'res_id': invoice.id,
                    'res_model': invoice._name,
                    'type': 'binary',
                    'raw': json.dumps(dict(request=eta_invoice)),
                    'mimetype': 'application/json',
                    'description': _('Egyptian Tax authority JSON invoice generated for %s.', invoice.name),
                })
            invoice.l10n_eg_eta_json_doc_id = attachment.id
        return drive_id.action_sign_invoices(invoices)

    def action_get_eta_invoice_pdf(self):
        """ This is a pdf with the structure from the government.  While we can use our own format,
        some clients appreciate this to verify that all the data is there in case of confusion."""
        self.ensure_one()
        eta_invoice_pdf = self.env['account.edi.format']._l10n_eg_get_eta_invoice_pdf(self)
        if eta_invoice_pdf.get('error', False):
            _logger.warning('PDF Content Error:  %s.', eta_invoice_pdf.get('error'))
            return
        self.with_context(no_new_invoice=True).message_post(body=_('ETA invoice has been received'),
                                                            attachments=[('ETA invoice of %s.pdf' % self.name,
                                                                          eta_invoice_pdf.get('data'))])

    def _l10n_eg_edi_exchange_currency_rate(self):
        """ Calculate the rate based on the balance and amount_currency, so we recuperate the one used at the time"""
        self.ensure_one()
        from_currency = self.currency_id
        to_currency = self.company_id.currency_id
        if from_currency != to_currency and self.invoice_line_ids:
            amount_currency = self.invoice_line_ids[0].amount_currency
            if not float_is_zero(amount_currency, precision_rounding=from_currency.rounding):
                return abs(self.invoice_line_ids[0].balance / amount_currency)
        return 1.0

```

## File: models\eta_activity_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models, fields, api
from odoo.osv import expression


class EtaActivityType(models.Model):
    _name = 'l10n_eg_edi.activity.type'
    _description = 'ETA code for activity type'

    name = fields.Char(required=True, translate=True)
    code = fields.Char(required=True)

    @api.model
    def _name_search(self, name='', args=None, operator='ilike', limit=100, name_get_uid=None):
        args = args or []
        if operator == 'ilike' and not(name or '').strip():
            domain = []
        else:
            domain = ['|', ('name', operator, name), ('code', operator, name)]
        return self._search(expression.AND([domain, args]), limit=limit, access_rights_uid=name_get_uid)

```

## File: models\eta_thumb_drive.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import hashlib
import json

import pytz

from asn1crypto import cms, core, x509, algos, tsp

from odoo import models, fields, _
from odoo.exceptions import ValidationError


class EtaThumbDrive(models.Model):
    _name = 'l10n_eg_edi.thumb.drive'
    _description = 'Thumb drive used to sign invoices in Egypt'

    user_id = fields.Many2one('res.users', required=True, default=lambda self: self.env.user)
    company_id = fields.Many2one('res.company', required=True, default=lambda self: self.env.company)
    certificate = fields.Binary('ETA Certificate')
    pin = fields.Char('ETA USB Pin', required=True)
    access_token = fields.Char(required=True)

    _sql_constraints = [
        ('user_drive_uniq', 'unique (user_id, company_id)', 'You can only have one thumb drive per user per company!'),
    ]

    def action_sign_invoices(self, invoice_ids):
        self.ensure_one()
        sign_host = self._get_host()

        to_sign_dict = dict()
        for invoice_id in invoice_ids:
            eta_invoice = json.loads(invoice_id.l10n_eg_eta_json_doc_id.raw)['request']
            signed_attrs = self._generate_signed_attrs__(eta_invoice, invoice_id.l10n_eg_signing_time)
            to_sign_dict[invoice_id.id] = base64.b64encode(signed_attrs.dump()).decode()

        return {
            'type': 'ir.actions.client',
            'tag': 'action_post_sign_invoice',
            'params': {
                'sign_host': sign_host,
                'access_token': self.access_token,
                'pin': self.pin,
                'drive_id': self.id,
                'invoices': json.dumps(to_sign_dict)
            }
        }

    def action_set_certificate_from_usb(self):
        self.ensure_one()
        sign_host = self._get_host()

        return {
            'type': 'ir.actions.client',
            'tag': 'action_get_drive_certificate',
            'params': {
                'sign_host': sign_host,
                'access_token': self.access_token,
                'pin': self.pin,
                'drive_id': self.id
            }
        }

    def set_certificate(self, certificate):
        """ This is called from the browser to set the certificate"""
        self.ensure_one()
        self.certificate = certificate.encode()
        return True

    def set_signature_data(self, invoices):
        """ This is called from the browser with the signed data from the local server """
        invoices = json.loads(invoices)
        for key, value in invoices.items():
            invoice_id = self.env['account.move'].browse(int(key))
            eta_invoice_json = json.loads(invoice_id.l10n_eg_eta_json_doc_id.raw)

            signature = self._generate_cades_bes_signature(eta_invoice_json['request'], invoice_id.l10n_eg_signing_time,
                                                           base64.b64decode(value))

            eta_invoice_json['request']['signatures'] = [{'signatureType': 'I', 'value': signature}]
            invoice_id.l10n_eg_eta_json_doc_id.raw = json.dumps(eta_invoice_json)
            invoice_id.l10n_eg_is_signed = True
        return True

    def _get_host(self):
        # It should be on the loopback address or with a fully valid https host
        # in order to be an exception to the mixed-content restrictions
        sign_host = self.env['ir.config_parameter'].sudo().get_param('l10n_eg_eta.sign.host', 'http://localhost:8069')
        if not sign_host:
            raise ValidationError(_('Please define the host of sign tool.'))
        return sign_host

    def _serialize_for_signing(self, eta_inv):
        if not isinstance(eta_inv, dict):
            return json.dumps(str(eta_inv), ensure_ascii=False)

        canonical_str = []
        for key, value in eta_inv.items():
            if not isinstance(value, list):
                canonical_str.append(json.dumps(key, ensure_ascii=False).upper())
                canonical_str.append(self._serialize_for_signing(value))
            else:
                canonical_str.append(json.dumps(key, ensure_ascii=False).upper())
                for elem in value:
                    canonical_str.append(json.dumps(key, ensure_ascii=False).upper())
                    canonical_str.append(self._serialize_for_signing(elem))
        return ''.join(canonical_str)

    def _generate_signed_attrs__(self, eta_invoice, signing_time):
        cert = x509.Certificate.load(base64.b64decode(self.certificate))
        data = hashlib.sha256(self._serialize_for_signing(eta_invoice).encode()).digest()
        return cms.CMSAttributes([
            cms.CMSAttribute({
                'type': cms.CMSAttributeType('content_type'),
                'values': ('digested_data',),
            }),
            cms.CMSAttribute({
                'type': cms.CMSAttributeType('message_digest'),
                'values': (data,),
            }),
            cms.CMSAttribute({
                'type': tsp.CMSAttributeType('signing_certificate_v2'),
                'values': ({
                               'certs': (tsp.ESSCertIDv2({
                                   'hash_algorithm': algos.DigestAlgorithm({'algorithm': 'sha256'}),
                                   'cert_hash': hashlib.sha256(cert.dump()).digest()
                               }),)
                           },),
            }),
            cms.CMSAttribute({
                'type': cms.CMSAttributeType('signing_time'),
                'values': (
                cms.Time({'utc_time': core.UTCTime(signing_time.replace(tzinfo=pytz.UTC))}),)
            }),
        ])

    def _generate_signer_info__(self, eta_invoice, signing_time, signature=False):
        cert = x509.Certificate.load(base64.b64decode(self.certificate))
        signer_info = {
            'version': 'v1',
            'sid': cms.SignerIdentifier({
                'issuer_and_serial_number': cms.IssuerAndSerialNumber({
                    'issuer': cert.issuer,
                    'serial_number': cert.serial_number,
                }),
            }),
            'digest_algorithm': algos.DigestAlgorithm({'algorithm': 'sha256'}),
            'signature_algorithm': algos.SignedDigestAlgorithm({
                'algorithm': 'sha256_rsa'
            }),
            'signed_attrs': self._generate_signed_attrs__(eta_invoice, signing_time)
        }
        if signature:
            signer_info['signature'] = signature
        return signer_info

    def _generate_cades_bes_signature(self, eta_invoice, signing_time, signature):
        cert = x509.Certificate.load(base64.b64decode(self.certificate))
        signed_data = {
            'version': 'v3',
            'digest_algorithms': cms.DigestAlgorithms((
                algos.DigestAlgorithm({'algorithm': 'sha256'}),
            )),
            'encap_content_info': {
                'content_type': 'digested_data',
            },
            'certificates': [cert],
            'signer_infos': [
                self._generate_signer_info__(eta_invoice, signing_time, signature),
            ],
        }
        content_info = cms.ContentInfo({'content_type': cms.ContentType('signed_data'), 'content': cms.SignedData(signed_data)})
        return base64.b64encode(content_info.dump()).decode()

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import api, fields, models


class ProductTemplate(models.Model):
    _inherit = "product.template"

    l10n_eg_eta_code = fields.Char('ETA Item code', compute='_compute_l10n_eg_eta_code',
                                   inverse='_set_l10n_eg_eta_code',
                                   help="This can be an EGS or GS1 product code, which is needed for the e-invoice.  "
                                        "The best practice however is to use that code also as barcode and in that case, "
                                        "you should put it in the Barcode field instead and leave this field empty.")

    @api.depends('product_variant_ids.l10n_eg_eta_code')
    def _compute_l10n_eg_eta_code(self):
        self.l10n_eg_eta_code = False
        for template in self:
            if len(template.product_variant_ids) == 1:
                template.l10n_eg_eta_code = template.product_variant_ids.l10n_eg_eta_code

    def _set_l10n_eg_eta_code(self):
        if len(self.product_variant_ids) == 1:
            self.product_variant_ids.l10n_eg_eta_code = self.l10n_eg_eta_code

    @api.model_create_multi
    def create(self, vals_list):
        templates = super().create(vals_list)

        for template, vals in zip(templates, vals_list):
            related_vals = {}
            if vals.get('l10n_eg_eta_code'):
                related_vals['l10n_eg_eta_code'] = vals['l10n_eg_eta_code']
            if related_vals:
                template.write(related_vals)

        return templates


class ProductProduct(models.Model):
    _inherit = "product.product"

    l10n_eg_eta_code = fields.Char('ETA Code', copy=False,
                                   help="This can be an EGS or GS1 product code, which is needed for the e-invoice.  "
                                        "The best practice however is to use that code also as barcode and in that case, "
                                        "you should put it in the Barcode field instead and leave this field empty. ")

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models, fields


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_eg_client_identifier = fields.Char('ETA Client ID', groups="base.group_erp_manager")
    l10n_eg_client_secret = fields.Char('ETA Secret', groups="base.group_erp_manager")
    l10n_eg_production_env = fields.Boolean('In Production Environment')
    l10n_eg_invoicing_threshold = fields.Float('Invoicing Threshold', default=0.0,
                                               help="Threshold at which you are required to give the VAT number "
                                                    "of the customer. ")

```

## File: models\res_config_settings.py

```python
from odoo import models, fields

class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_eg_client_identifier = fields.Char(related='company_id.l10n_eg_client_identifier', readonly=False)
    l10n_eg_client_secret = fields.Char(related='company_id.l10n_eg_client_secret', readonly=False)
    l10n_eg_production_env = fields.Boolean(related='company_id.l10n_eg_production_env', readonly=False)
    l10n_eg_invoicing_threshold = fields.Float(related='company_id.l10n_eg_invoicing_threshold', readonly=False)

```

## File: models\res_currency_rate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models, api, _
from odoo.tools import float_compare


class ResCurrencyRate(models.Model):
    _inherit = 'res.currency.rate'

    @api.onchange('company_rate')
    def _onchange_rate_warning(self):
        # We send the ETA a rate that is 5 decimal accuracy, so to ensure consistency, Odoo should also operate with 5 decimal accuracy rate
        if (
            self.company_id.account_fiscal_country_id.code == 'EG' and
            float_compare(self.inverse_company_rate, round(self.inverse_company_rate, 5), precision_digits=10) != 0
            ):
            return {
                'warning': {
                    'title': _("Warning for %s", self.currency_id.name),
                    'message': _(
                        "Please make sure that the EGP per unit is within 5 decimal accuracy.\n"
                        "Higher decimal accuracy might lead to inconsistency with the ETA invoicing portal!"
                    )
                }
            }
        return super()._onchange_rate_warning()

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models, fields, api


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_eg_building_no = fields.Char('Building No.')

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['l10n_eg_building_no']

    def _address_fields(self):
        return super()._address_fields() + ['l10n_eg_building_no']

```

## File: models\uom_uom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class UomCode(models.Model):
    _name = 'l10n_eg_edi.uom.code'
    _description = 'ETA code for the unit of measures'

    name = fields.Char(required=True, translate=True)
    code = fields.Char(required=True)


class UomUom(models.Model):
    _inherit = 'uom.uom'

    l10n_eg_unit_code_id = fields.Many2one('l10n_eg_edi.uom.code', string='ETA Unit Code',
                                           help='This is the type of unit according to egyptian tax authority')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import eta_activity_type
from . import uom_uom
from . import res_company
from . import res_partner
from . import account_move
from . import account_edi_format
from . import account_journal
from . import eta_thumb_drive
from . import res_currency_rate
from . import product_template
from . import res_config_settings

```

## File: security\eta_thumb_drive_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="eta_thumb_drive_modify_own" model="ir.rule">
        <field name="name">Only see/modify own thumb drive</field>
        <field name="model_id" ref="l10n_eg_edi_eta.model_l10n_eg_edi_thumb_drive"/>
        <field name="domain_force">[('user_id', '=', user.id)]</field>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_eg_edi_activity_type,access_l10n_eg_edi_activity_type,model_l10n_eg_edi_activity_type,base.group_user,1,0,0,0
access_l10n_eg_edi_uom_code,access_l10n_eg_edi_uom_code,model_l10n_eg_edi_uom_code,base.group_user,1,0,0,0
access_l10n_eg_edi_eta_thumb_drive,access_l10n_eg_edi_eta_thumb_drive,model_l10n_eg_edi_thumb_drive,account.group_account_manager,1,1,1,1

```

## File: static\src\js\sign_invoice.js

```javascript
odoo.define('l10n_eg_edi_eta.action_post_sign_invoice', function (require) {
    const core = require('web.core');
    const ajax = require('web.ajax');
    const Dialog = require('web.Dialog');
    var rpc = require('web.rpc');
    var _t = core._t;

    function get_drive_error(value) {
        switch(value) {
           case 'no_pykcs11': return _t("Missing library - Please make sure that PyKCS11 is correctly installed on the local proxy server");
           case 'missing_dll': return _t("Missing Dependency - If you are using Windows, make sure eps2003csp11.dll is correctly installed. You can download it here: https://www.egypttrust.com/en/downloads/other-drivers. If you are using Linux or macOS, please install OpenSC");
           case 'no_drive': return _t("No drive found - Make sure the thumb drive is correctly inserted");
           case 'multiple_drive': return _t("Multiple drive detected - Only one secure thumb drive can be inserted at the same time");
           case 'system_unsupported': return _t("System not supported");
           case 'unauthorized': return _t("Unauthorized");
        }
        return _t("Unexpected error:") + value;

    }

    async function action_get_drive_certificate(parent, {params}) {
        const host = params.sign_host;
        const drive_id = params.drive_id;
        delete params.sign_host;
        delete params.drive_id;
        await ajax.post(host + '/hw_l10n_eg_eta/certificate', params).then(function (res) {
            const res_obj = JSON.parse(res);
            if (res_obj.error) {
                Dialog.alert(this, get_drive_error(res_obj.error));
            } else if (res_obj.certificate) {
                rpc.query({
                    model: 'l10n_eg_edi.thumb.drive',
                    method: 'set_certificate',
                    args: [[drive_id], res_obj.certificate],
                }).then(function () {
                    parent.services.action.doAction({
                        'type': 'ir.actions.client',
                        'tag': 'reload',
                    });
                }, function () {
                    Dialog.alert(this, _t("Error trying to connect to Odoo. Check your internet connection"));
                })

            } else {
                Dialog.alert(this, _t('An unexpected error has occurred'));
            }
        }, function () {
            Dialog.alert(this, _t("Error trying to connect to the middleware. Is the middleware running?"));
        })
    }

    async function action_post_sign_invoice(parent, {params}) {
        const host = params.sign_host;
        const drive_id = params.drive_id;
        delete params.sign_host;
        delete params.drive_id;
        await ajax.post(host + '/hw_l10n_eg_eta/sign', params).then(function (res) {
            const res_obj = JSON.parse(res);
            if (res_obj.error) {
                Dialog.alert(this, get_drive_error(res_obj.error));
            } else if (res_obj.invoices) {
                rpc.query({
                    model: 'l10n_eg_edi.thumb.drive',
                    method: 'set_signature_data',
                    args: [[drive_id], res_obj.invoices],
                }).then(function () {
                    parent.services.action.doAction({
                        'type': 'ir.actions.client',
                        'tag': 'reload',
                    });
                }, function () {
                    Dialog.alert(this, _t("Error trying to connect to Odoo. Check your internet connection"));
                })

            } else {
                Dialog.alert(this, _t('An unexpected error has occurred'));
            }
        }, function () {
            Dialog.alert(this, _t("Error trying to connect to the middleware. Is the middleware running?"));
        })
    }

    core.action_registry.add('action_get_drive_certificate', action_get_drive_certificate);
    core.action_registry.add('action_post_sign_invoice', action_post_sign_invoice);

    return action_post_sign_invoice;
});

```

## File: views\account_journal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_journal_form_inherit_l10n_eg_edi" model="ir.ui.view">
            <field name="name">account.journal.form.inherit.l10n_eg_edi</field>
            <field name="model">account.journal</field>
            <field name="priority">10</field>
            <field name="inherit_id" ref="account.view_account_journal_form"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='advanced_settings']/group" position="inside">
                    <group string="Egyptian ETA settings" attrs="{'invisible': ['|', ('type', '!=', 'sale'), ('country_code', '!=', 'EG')]}">
                        <field name="l10n_eg_branch_id"/>
                        <field name="l10n_eg_activity_type_id"/>
                        <field name="l10n_eg_branch_identifier"/>
                    </group>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\account_move_view.xml

```xml
<odoo>
    <data>
        <record id="action_sign_invoices" model="ir.actions.server">
            <field name="name">Sign invoices</field>
            <field name="type">ir.actions.server</field>
            <field name="state">code</field>
            <field name="model_id" ref="account.model_account_move"/>
            <field name="binding_model_id" ref="account.model_account_move"/>
            <field name="code">
                action = records.action_post_sign_invoices()
            </field>
        </record>


        <record id="view_move_form_inherit" model="ir.ui.view">
            <field name="name">view_move_form_inherit</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="priority" eval="40"/>
            <field name="arch" type="xml">
                <xpath expr="//header/button[@name='action_post']" position="after">
                    <button name="action_post_sign_invoices" type="object"
                            class="oe_highlight"
                            groups="account.group_account_manager"
                            string="Sign Invoice"
                            attrs="{'invisible': ['|', '|', ('country_code', '!=', 'EG'), ('l10n_eg_is_signed', '=', True), ('state', '!=', 'posted')]}"/>
                </xpath>
                <notebook position="inside">
                    <page string="ETA E-Invoice" attrs="{'invisible': [('country_code', '!=', 'EG')]}">
                        <group>
                            <group>
                                <field name="l10n_eg_uuid" readonly="1"/>
                                <field name="l10n_eg_submission_number" readonly="1"/>
                            </group>
                            <group>
                                <field name="l10n_eg_eta_json_doc_id" readonly="1" invisible="1"/>
                                <field name="l10n_eg_is_signed" invisible="1"/>
                            </group>
                            <group>
                                <button name="action_get_eta_invoice_pdf" type="object"
                                        groups="account.group_account_invoice"
                                        class="oe_link"
                                        icon="fa-clone"
                                        string="Get ETA Invoice PDF"
                                        attrs="{'invisible': [('edi_state', '!=', 'sent')]}"/>
                            </group>
                        </group>
                    </page>
                </notebook>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\eta_thumb_drive.xml

```xml
<odoo>
    <data>
        <record id="view_l10n_eg_edi_thumb_drive_tree" model="ir.ui.view">
            <field name="name">view_l10n_eg_edi_thumb_drive_tree</field>
            <field name="model">l10n_eg_edi.thumb.drive</field>
            <field name="arch" type="xml">
                <tree editable="top">
                    <field name="user_id" invisible="1"/>
                    <field name="certificate" invisible="1"/>
                    <field name="company_id"/>
                    <field name="pin" password="True"/>
                    <field name="access_token" password="True"/>
                    <button name="action_set_certificate_from_usb" type="object" string="Get certificate" attrs="{'invisible': [('certificate', '!=', False)]}"/>
                </tree>
            </field>
        </record>

        <record id="action_eta_thumb_drive_tree" model="ir.actions.act_window">
            <field name="name">Thumb Drive</field>
            <field name="res_model">l10n_eg_edi.thumb.drive</field>
            <field name="view_mode">tree</field>
        </record>

        <menuitem id="account_eta_menu" name="ETA" groups="account.group_account_manager" parent="account.menu_finance_configuration" sequence="99">
            <menuitem id="menu_action_eta_thumb_drive_tree" action="action_eta_thumb_drive_tree" sequence="1"/>
        </menuitem>
    </data>
</odoo>

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="product_template_only_form_view_inherit_l10n_eg_eta_edi" model="ir.ui.view">
            <field name="name">product.template.form.l10n_eg_eta_edi</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_only_form_view"/>
            <field name="arch" type="xml">
                <page name="invoicing" position="inside">
                    <group name="l10n_eg_eta_edi" string="Egyptian Electronic Invoicing" attrs="{'invisible': [('product_variant_count', '>', 1)]}">
                        <field name="l10n_eg_eta_code" help="ETA Field for GS1/EGS product codes. Please use the barcode field to store GS1/EGS ETA code if possible"/>
                    </group>
                </page>
            </field>
        </record>

        <record id="product_normal_form_view_inherit_l10n_eg_eta_edi" model="ir.ui.view">
            <field name="name">product.product.form.l10n_eg_eta_edi</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_normal_form_view" />
            <field name="arch" type="xml">
                <page name="invoicing" position="inside">
                    <group name="l10n_eg_eta_edi" string="Egyptian Electronic Invoicing">
                        <field name="l10n_eg_eta_code" help="ETA Field for GS1/EGS product codes. Please use the barcode field to store GS1/EGS ETA code if possible"/>
                    </group>
                </page>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\report_invoice.xml

```xml
<odoo>
    <data>
        <template id="egyptian_invoice" inherit_id="account.report_invoice_document">
            <xpath expr="//div[@id='qrcode']" position="after">
                <div id="qrcode_eg" t-if="o.l10n_eg_qr_code">
                    <p>
                        <strong class="text-center">ETA QR Code</strong>
                        <img style="display:block;"
                             t-att-src="'/report/barcode/?barcode_type=%s&amp;value=%s&amp;width=%s&amp;height=%s'%('QR', o.l10n_eg_qr_code, 130, 130)"/>
                    </p>
                </div>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="res_config_settings_view_form">
        <field name="name">res.config.settings.view.form.inherit.l10n_eg_edi_eta</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@data-key='account']/div" position="after">
                <h2 attrs="{'invisible':[('country_code', '!=', 'EG')]}">ETA E-Invoicing Settings</h2>
                <div class="row mt16 o_settings_container" name="egyption_eta_edi" attrs="{'invisible':[('country_code', '!=', 'EG')]}">
                    <div class="col-xs-12 col-md-6 o_setting_box">
                        <div class="o_setting_left_pane"/>
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">ETA API Integration</span>
                            <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                            <div class="text-muted">
                                    Enter your API credentials to enable ETA E-Invoicing.
                                    <field name="country_code" invisible="1"/>
                            </div>
                            <div class="content-group">
                                <div class="row mt16">
                                    <label for="l10n_eg_production_env" class="col-lg-6 o_light_label"/>
                                    <field name="l10n_eg_production_env" help="Check to start sending invoices to your e-invoicing production environment"/>
                                </div>
                                <div class="row">
                                    <label for="l10n_eg_client_identifier" class="col-lg-4 o_light_label"/>
                                    <field name="l10n_eg_client_identifier" help="The client ID retrieved from the ETA e-invoicing portal"/>
                                </div>
                                <div class="row">
                                    <label for="l10n_eg_client_secret" class="col-lg-4 o_light_label"/>
                                    <field name="l10n_eg_client_secret" password="True" help="The secret key provided by the ETA. You can input client secret 1 or 2"/>
                                </div>
                                <div class="row">
                                    <label for="l10n_eg_invoicing_threshold" class="col-lg-4 o_light_label"/>
                                    <field name="l10n_eg_invoicing_threshold" help="Set the threshold amount for invoices that won't require the VAT ID of individuals when invoicing"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\uom_uom_view.xml

```xml
<odoo>
    <data>
        <record id="product_uom_form_view_inherit" model="ir.ui.view">
            <field name="name">product_uom_form_view_inherit</field>
            <field name="model">uom.uom</field>
            <field name="inherit_id" ref="uom.product_uom_form_view"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='uom_type']" position="after">
                    <field name="l10n_eg_unit_code_id"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

