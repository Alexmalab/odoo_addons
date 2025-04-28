# Odoo Module: l10n_br

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2009  Renato Lima - Akretion

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2009  Renato Lima - Akretion

{
    'name': 'Brazilian - Accounting',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Base module for the Brazilian localization
==========================================

This module consists of:

 - Generic Brazilian chart of accounts
 - Brazilian taxes such as:

        - IPI
        - ICMS
        - PIS
        - COFINS
        - ISS
        - IR
        - IRPJ
        - CSLL

The field tax_discount has also been added in the account.tax.template and
account.tax objects to allow the proper computation of some Brazilian VATs
such as ICMS. The chart of account creation wizard has been extended to
propagate those new data properly.

It's important to note however that this module lack many implementations to
use Odoo properly in Brazil. Those implementations (such as the electronic
fiscal Invoicing which is already operational) are brought by more than 15
additional modules of the Brazilian Launchpad localization project
https://launchpad.net/openerp.pt-br-localiz and their dependencies in the
extra addons branch. Those modules aim at not breaking with the remarkable
Odoo modularity, this is why they are numerous but small. One of the
reasons for maintaining those modules apart is that Brazilian Localization
leaders need commit rights agility to complete the localization as companies
fund the remaining legal requirements (such as soon fiscal ledgers,
accounting SPED, fiscal SPED and PAF ECF that are still missing as September
2011). Those modules are also strictly licensed under AGPL V3 and today don't
come with any additional paid permission for online use of 'private modules'.
""",
    'author': 'Akretion, Odoo Brasil',
    'website': 'http://openerpbrasil.org',
    'depends': ['account'],
    'data': [
        'data/l10n_br_chart_data.xml',
        'data/account.account.template.csv',
        'data/account_tax_group_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'views/account_view.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,note,user_type_id:id,reconcile,chart_template_id:id
account_template_101010300,1.01.01.03.00,Recursos no Exterior Decorrentes de Exportação,"Contas que registram movimentação de recursos em instituições financeiras no exterior, nos termos do art. 1o. da Lei no 11.371/2006.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101010400,1.01.01.04.00,Contas Bancárias – Subvenções,"Contas que registram disponibilidades, nas instituições imunes ou isentas, de recursos de aplicações vinculadas ao objeto das subvenções, mantidas em instituições financeiras.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101010500,1.01.01.05.00,Contas Bancárias – Doações,"Contas que registram disponibilidades, nas instituições imunes ou isentas, de recursos de aplicações vinculadas ao objeto das doações, mantidas em instituições financeiras.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101010600,1.01.01.06.00,Contas Bancárias – Outros Recursos Sujeitos a Restrições,"Contas que registram disponibilidades, nas instituições imunes ou isentas, de outros recursos sujeitos a restrições, mantidas em instituições financeiras.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101010701,1.01.01.07.01,Valores Mobiliários - Mercado de Capitais Interno,"Contas que registram as aplicações no mercado de capitais do Brasil, de recursos de livre movimentação, cujo vencimento ou resgate venha a ocorrer no curso do ano-calendário subseqüente.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101010702,1.01.01.07.02,Valores Mobiliários - Mercado de Capitais Externo,"Contas que registram as aplicações no mercado de capitais do exterior, de recursos de livre movimentação, cujo vencimento ou resgate venha a ocorrer no curso do ano-calendário subseqüente.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101010800,1.01.01.08.00,Valores Mobiliários – Aplicações de Subvenções  ,"Contas que correspondem, nas instituições imunes ou isentas, às aplicações financeiras de recursos oriundos de subvenções.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101010900,1.01.01.09.00,Valores Mobiliários – Aplicações de Doações,"Contas que correspondem, nas instituições imunes ou isentas, às aplicações financeiras de recursos oriundos de doações.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101011000,1.01.01.10.00,Valores Mobiliários – Aplicações de Outros Recursos Sujeitos a Restrições,"Contas que correspondem, nas instituições imunes ou isentas, às aplicações financeiras de outros recursos sujeitos a restrições.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101011100,1.01.01.11.00,Outras, ,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030101,1.01.03.01.01,Mercadorias para Revenda,"Contas que registram o valor do saldo das contas de estoques de mercadorias para revenda, na data de apuração dos resultados. Observar, quanto aos estoques, as orientações contidas na Instrução Normativa SRF no 51, de 1978, e no PN CST no 6, de 1979. ",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030102,1.01.03.01.02,Insumos (materiais diretos),"Contas que registram o valor do saldo das contas de estoques de matérias primas e materiais diretos, na data de apuração dos resultados. Observar, quanto aos estoques, as orientações contidas na Instrução Normativa SRF no 51, de 1978, e no PN CST no 6, de 1979. ",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030103,1.01.03.01.03,Produtos em Elaboração,"Contas que registram o valor do saldo das contas de estoques de produtos em elaboração, na data de apuração dos resultados. Observar, quanto aos estoques, as orientações contidas na Instrução Normativa SRF no 51, de 1978, e no PN CST no 6, de 1979. ",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030104,1.01.03.01.04,Produtos Acabados,"Contas que registram o valor do saldo das contas de estoques de produtos acabados, na data de apuração dos resultados. Observar, quanto aos estoques, as orientações contidas na Instrução Normativa SRF no 51, de 1978, e no PN CST no 6, de 1979. ",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030105,1.01.03.01.05,Serviços em andamento,"Contas que registram o valor do saldo das contas de serviços em andamento, na data de apuração dos resultados. Observar, quanto aos estoques, as orientações contidas na Instrução Normativa SRF no 51, de 1978, e no PN CST no 6, de 1979. ",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030106,1.01.03.01.06,Insumos Agropecuários,"Contas que registram, nas empresas com atividade rural, o valor do saldo das contas de insumos agropecuários, na data de apuração dos resultados.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030107,1.01.03.01.07,Produtos Agropecuários em Formação,"Contas que registram, nas empresas com atividade rural, o valor do saldo das contas de produtos agropecuários em formação, na data de apuração dos resultados.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030108,1.01.03.01.08,Produtos Agropecuários Acabados,"Contas que registra, nas empresas com atividade rural, o valor do saldo das contas de estoques de produtos agropecuários acabados, na data de apuração do resultado.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030200,1.01.03.02.00,Imóveis Destinados à Venda,Contas utilizadas pela pessoa jurídica que exerce atividade imobiliária para indicar o estoque de imóveis destinados à venda existente na data da apuração dos resultados. Atenção: as construções em andamento de imóveis destinados à venda devem ser incluídas na conta Construções em Andamento de Imóveis Destinados à Venda,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030201,1.01.03.02.01,Construções em Andamento de Imóveis Destinados à Venda,Contas utilizadas pela pessoa jurídica que exerce atividade imobiliária para indicar os imóveis em construção para futura comercialização,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030300,1.01.03.03.00,Estoques Destinados à Doação,"Contas que registram, nas instituições imunes ou isentas, estoques destinados à doação.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101030400,1.01.03.04.00,Outras, ,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050100,1.01.05.01.00,Adiantamentos a Fornecedores ,Contas que registram aos adiantamentos feitos a fornecedores de matérias-primas ou mercadorias para revenda.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050200,1.01.05.02.00,Clientes CIRCULANTE,Contas que registram as contas a receber com vencimento até o final do ano-calendário subseqüente.,account.data_account_type_receivable,TRUE,l10n_br_account_chart_template
account_template_101050201,1.01.05.02.01,Clientes CIRCULANTE (PoS),Contas que registram as contas a receber com vencimento até o final do ano-calendário subseqüente.,account.data_account_type_receivable,TRUE,l10n_br_account_chart_template
account_template_101050300,1.01.05.03.00,Créditos Fiscais CSLL – Diferenças Temporárias e Base de Cálculo Negativa,"As companhias abertas, obrigatoriamente, devem informar, nestas contas, o valor dos créditos fiscais com realização no exercício seguinte e das diferenças temporárias, inclusive as decorrentes da base de cálculo negativa, relativos à CSLL, conforme Deliberação CVM no 273, de 20 de agosto de 1998.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050400,1.01.05.04.00,Créditos Fiscais IRPJ – Diferenças Temporárias e Prejuízos Fiscais,"As companhias abertas, obrigatoriamente, devem informar, nestas contas, o valor dos créditos fiscais com realização no exercício seguinte e das diferenças temporárias, inclusive as decorrentes dos prejuízos fiscais, relativos ao IRPJ, conforme Deliberação CVM nº 273, de 20 de agosto de 1998.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050501,1.01.05.05.01,Imposto de Renda a Recuperar,Contas correspondentes ao Imposto de REnda a recuperar no final do período de apuração.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050502,1.01.05.05.02,IPI a Recuperar,Contas correspondentes ao IPI a recuperar no final do período de apuração.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050503,1.01.05.05.03,PIS e COFINS a Recuperar,Contas correspondentes ao PIS e à Cofins a recuperar no final do período de apuração.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050504,1.01.05.05.04,CSLL a Recuperar,Contas correspondentes  à CSLL a recuperar no final do período de apuração.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050505,1.01.05.05.05,ICMS e Contribuições a Recuperar,Contas correspondentes ao ICMS a recuperar no final do período de apuração.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050506,1.01.05.05.06,Tributos Municipais a Recuperar,Contas correspondentes a tributos municipais a recuperar no final do período de apuração.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050590,1.01.05.05.90,Outros Impostos e Contribuições a Recuperar,Contas correspondentes a outros impostos a recuperar no final do período de apuração.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050600,1.01.05.06.00,Créditos por Contribuições e Doações,"Contas que registram, nas instituições imunes ou isentas, créditos por contribuições ou doações.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101050700,1.01.05.07.00,Outras, ,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101070100,1.01.07.01.00,Despesas do Exercício Seguinte,"Contas correspondentes a pagamentos antecipados, cujos benefícios ou prestação de serviços à pessoa jurídica ocorrerão durante o exercício seguinte. São valores relativos a despesas que efetivamente pertencem ao exercício seguinte.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101070200,1.01.07.02.00,Outras Contas,"Incluir, dentre outras, a soma das contas/subcontas do Circulante que registram, dentre outras, a correção monetária relativa à diferença, em relação ao ano de 1990, entre o IPC e o BTN Fiscal, na forma estabelecida nos arts. 32 e 33 do Decreto no 332, de 1991.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101090101,1.01.09.01.01,(-) Duplicatas Descontadas,"Contas que registram parcelas a serem subtraídas do circulante, correspondentes a valores das duplicatas descontadas que retificam este grupo",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101090103,1.01.09.01.03,(-) Provisões para Créditos de Liquidação Duvidosa,"Contas que registram parcelas a serem subtraídas do circulante, correspondentes a valores das provisões para créditos de liquidação duvidosa que retificam este grupo.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101090105,1.01.09.01.05,(-) Provisão para Ajuste do Estoque ao Valor de Mercado,"Contas que registram parcelas a serem subtraídas do circulante, correspondentes a valores das provisões para ajuste do estoque ao valor de mercado que retificam este grupo.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101090107,1.01.09.01.07,(-) Provisões para Ajuste ao Valor Provável de Realização,"Contas que registram parcelas a serem subtraídas do circulante, correspondentes a valores das provisões para ajuste do estoque ao valor provável de realização que retificam este grupo.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_101090190,1.01.09.01.90,(-) Outras Contas Retificadoras,Contas que registram parcelas a serem subtraídas do circulante que não possam ser classificadas nos itens precedentes.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107000100,1.07.00.01.00,Clientes NÃO CIRCULANTE,"Contas que registram os créditos a receber de terceiros, relativos a eventuais contas de clientes, títulos a receber, adiantamentos, etc., com prazo de recebimento posterior ao exercício seguinte à data do balanço.",account.data_account_type_receivable,TRUE,l10n_br_account_chart_template
account_template_107000200,1.07.00.02.00,Créditos com Pessoas Ligadas (Físicas/Jurídicas),"Contas correspondentes a vendas, adiantamentos ou empréstimos a sociedades coligadas ou controladas, diretores, acionistas ou participantes da empresa, que não constituam negócios usuais na exploração do objeto social da pessoa jurídica.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107000300,1.07.00.03.00,Valores Mobiliários,"Contas correspondentes às aplicações em títulos com vencimento posterior ao exercício seguinte, e investimentos em outras sociedades que não tenham caráter permanente, inclusive os feitos com incentivos fiscais.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107000400,1.07.00.04.00,Depósitos Judiciais,"Contas que registram aos depósitos judiciais efetuados, a qualquer título, pendentes de decisão.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107000500,1.07.00.05.00,Créditos Fiscais CSLL – Diferenças Temporárias e Base de Cálculo Negativa,"As companhias abertas, obrigatoriamente, devem informar, nestas contas, o valor dos créditos fiscais com realização após o exercício seguinte e das diferenças temporárias, inclusive as decorrentes da base de cálculo negativa, relativos à CSLL, conforme Deliberação CVM no 273, de 1998.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107000600,1.07.00.06.00,Créditos Fiscais IRPJ – Diferenças Temporárias e Prejuízos Fiscais,"As companhias abertas, obrigatoriamente, devem informar, nestas contas, o valor dos créditos fiscais com realização após o exercício seguinte e das diferenças temporárias, inclusive as decorrentes dos prejuízos fiscais, relativos ao IRPJ, conforme Deliberação CVM no 273, de 1998.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107000700,1.07.00.07.00,Créditos por Contribuições e Doações,"Contas que registram, nas instituições imunes ou isentas, créditos por contribuições ou doações com vencimento após final do exercício subseqüente.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107000800,1.07.00.08.00,Outras Contas,"Contas que registram, entre outras, a soma das contas/subcontas do Realizável a Longo Prazo que registram a correção monetária relativa à diferença, em relação ao ano de 1990, entre o IPC e o BTN Fiscal, na forma estabelecida nos arts. 32 e 33 do Decreto no 332, de 1991.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107009000,1.07.00.90.00,(-) Duplicatas Descontadas,"Contas que registram parcelas a serem subtraídas do circulante, correspondentes a valores das duplicatas descontadas que retificam este grupo",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107009300,1.07.00.93.00,(-) Provisões para Créditos de Liquidação Duvidosa,"Contas que registram parcelas a serem subtraídas do circulante, correspondentes a valores das provisões para créditos de liquidação duvidosa que retificam este grupo.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107009500,1.07.00.95.00,(-) Provisões para Ajuste ao Valor Provável de Realização,"Contas que registram parcelas a serem subtraídas do circulante, correspondentes a valores das provisões para ajuste do estoque ao valor provável de realização que retificam este grupo.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107009700,1.07.00.97.00,(-) Outras Contas Retificadoras,Contas que registram parcelas a serem subtraídas do Realizável a Longo Prazo que não possam ser classificadas nos itens precedentes.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107010100,1.07.01.01.00,Participações Permanentes em Coligadas ou Controladas,"Contas que registram investimentos permanentes, na forma de participação em outras sociedades coligadas e/ou controladas, ainda que se trate de investimento não relevante.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107010200,1.07.01.02.00,Investimentos Decorrentes de Incentivos Fiscais,"Contas que registram os investimentos decorrentes de incentivos fiscais representados por ações novas da Embraer ou de empresas nacionais de informática ou por participação direta decorrente da troca do CI – Certificado de Investimento por ações pertencentes às carteiras de Fundos (Finor, Finam e Fiset). Inclui-se a aquisição de quotas representativas de direitos de comercialização sobre produção de obras audiovisuais cinematográficas brasileiras de produção independente, com projetos  previamente aprovados pelo Ministério da Cultura, realizada no mercado de capitais, em ativos previstos em lei e autorizados pela Comissão de Valores Mobiliários (CVM).",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107010300,1.07.01.03.00,Outros Investimentos,"Contas correspondentes aos direitos de qualquer natureza que não se destinem à manutenção da atividade da companhia ou da empresa e que não se classifiquem no ativo circulante ou realizável a longo prazo, tais como: o imóvel não utilizado na exploração ou na manutenção das atividades da empresa e que não se destine à revenda, e os recursos florestais destinados à proteção do solo ou à preservação da natureza, entre outros.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107010400,1.07.01.04.00,Ágios em Investimentos,"Contas correspondentes ao ágio por diferença de valor de mercado dos bens, por valor de rentabilidade futura, por fundo de comércio, intangíveis, ou outras razões econômicas.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107010500,1.07.01.05.00,Correção Monetária - Diferença IPC/BTNF (Lei no 8.200/1991),"Contas/subcontas dos investimentos que registram a correção monetária relativa à diferença, em relação ao ano de 1990, entre o IPC e o BTN Fiscal, na forma estabelecida nos arts. 32 e 33 do Decreto no 332, de 1991.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107010600,1.07.01.06.00,Correção Monetária Especial (Lei no 8.200/1991),"Contas/subcontas dos investimentos que registram a correção monetária especial, na forma do art. 44 do Decreto no 332, de 1991.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107010700,1.07.01.07.00,(-) Deságios e Provisão para Perdas Prováveis em Investimentos,"Contas que registram:
a) o deságio por diferença de valor de mercado dos bens, por valor de rentabilidade futura e por fundo de comércio, intangíveis, ou outras razões econômicas; 
b) o valor correspondente à provisão para perdas em investimentos registrados pelo método de custo e à provisão para perdas em investimentos avaliados pelo método da equivalência patrimonial, sendo que, neste último caso, deve ser informado somente o valor das perdas efetivas ou potenciais já previstas, mas não reconhecidas contabilmente pela coligada ou controlada.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107019000,1.07.01.90.00,Outras Contas,Contas que registram bens e direitos classificáveis em Investimentos que não possam ser classificadas nos itens precedentes.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107019700,1.07.01.97.00,(-) Outras Contas Retificadoras,Contas que registram parcelas a serem subtraídas de Investimentos que não possam ser classificadas nos itens precedentes.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040100,1.07.04.01.00,Terrenos,"Contas que registram os terrenos de propriedade da pessoa jurídica utilizados nas operações, ou seja, onde se localizam a fábrica, os depósitos, os escritórios, as filiais, as lojas, etc., inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens 
Atenção: o valor do terreno onde está em construção uma nova unidade que ainda não esteja em operação também deve ser informado nesta conta.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040200,1.07.04.02.00,Edifícios e Construções,"Contas que registram os edifícios, melhoramentos e obras integradas aos terrenos, e os serviços e instalações provisórias, necessários à construção e ao andamento das obras, tais como: limpeza do terreno, serviços topográficos, sondagens de reconhecimento, terraplenagem, e outras similares, inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens 
Atenção: As construções em andamento devem ser informadas na conta  Construções em Andamento.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040201,1.07.04.02.01, Construções em Andamento,"Contas que registram as construções em andamento de edifícios, melhoramentos e obras integradas aos terrenos, e os serviços e instalações provisórias, necessários à construção e ao andamento das obras, tais como: limpeza do terreno, serviços topográficos, sondagens de reconhecimento, terraplenagem, e outras similares, inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040300,1.07.04.03.00,"Equipamentos, Máquinas e Instalações Industriais","Contas que registram os equipamentos, máquinas e instalações industriais utilizados no processo de produção da pessoa jurídica, inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040400,1.07.04.04.00,Veículos,"Contas que registram os veículos de propriedade da pessoa jurídica. inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens
Atenção: Os veículos de uso direto na produção, como empilhadeiras e similares, devem ser informados na conta Equipamentos, Máquinas e Instalações Industriais.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040401,1.07.04.04.01,Embarcações,"Contas que registram as embarcações de propriedade da pessoa jurídica., inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040402,1.07.04.04.02,Aeronaves,"Contas que registram as aeronaves de propriedade da pessoa jurídica., inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040500,1.07.04.05.00,"Móveis, Utensílios e Instalações Comerciais","Contas que registram os móveis, utensílios e instalações comerciais., inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040600,1.07.04.06.00,Recursos Minerais,"Contas que registram os direitos de exploração de jazidas de minério, de pedras preciosas, e similares, inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040700,1.07.04.07.00,Florestamento e Reflorestamento,"Contas que registram os recursos florestais destinados à exploração dos respectivos frutos e ao corte para comercialização, consumo ou industrialização, inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040800,1.07.04.08.00,Direitos Contratuais de Exploração de Florestas,"Contas que registram os direitos contratuais de exploração de florestas com prazo de exploração superior a dois anos., inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107040900,1.07.04.09.00,Outras Imobilizações,"Contas que registram outras imobilizações, tais como: benfeitorias em propriedades arrendadas que se incorporam ao imóvel arrendado e revertem ao proprietário do imóvel ao final da locação, adiantamentos para inversões fixas, reprodutores, matrizes e as culturas permanentes da atividade rural, e similares, inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107041000,1.07.04.10.00,Correção Monetária - Diferença IPC/BTNF (Lei no 8.200/1991),"Contas/subcontas do imobilizado que registram a correção monetária relativa à diferença, em relação ao ano de 1990, entre o IPC e o BTN Fiscal, na forma estabelecida nos arts. 32 e 33 do Decreto no 332, de 1991.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107041100,1.07.04.11.00,Correção Monetária Especial (Lei no 8.200/1991),"Contas/subcontas do imobilizado que registram a correção monetária especial na forma do art. 44 do Decreto no 332, de 1991.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107041200,1.07.04.12.00,"(-) Depreciações, Amortizações e Quotas de Exaustão","Contas que registram as depreciações, amortizações e quotas de exaustão das contas do imobilizado.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107049000,1.07.04.90.00,(-) Outras Contas Redutoras do Imobilizado,"Outras contas redutoras do Imobilizado, inclusive a provisão para perda decorrente da análise de  recuperação (art. 183, §3º, da Lei 6.404/76) ",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107050100,1.07.05.01.00,Concessões,Contas que registram os custos com aquisição de concessões,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107050300,1.07.05.03.00,Marcas e Patentes,Contas que registram os custos com aquisição de marcas e patentes,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107050500,1.07.05.05.00,Direitos Autorais,Contas que registram os custos com aquisição de direitos autorais,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107050700,1.07.05.07.00,Fundo de Comércio,Contas que registram os custos com aquisição de fundos de comércio,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107050900,1.07.05.09.00,Software ou Programas de Computador,Contas que registram os custos com aquisição/desenvolvimento de programas de computador,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107051100,1.07.05.11.00,Franquias,Contas que registram os custos com aquisição de franquias,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107051300,1.07.05.13.00,Desenvolvimento de Produtos,Contas que registram os custos com o desenvolvimento de novos produtos,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107051500,1.07.05.15.00,Outras,Contas que registram os custos com aquisição de outros itens classificáveis no intangível,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107059000,1.07.05.90.00,(-) Amortização do Intangível,Contas correspondentes à amortização das contas do ativo intangível,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107059700,1.07.05.97.00,(-) Outras Contas Redutoras do Intangível,"Outras contas redutoras o intangível, inclusive a provisão para perda decorrente da análise de  recuperação (art. 183, §3º, da Lei 6.404/76) ",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107070100,1.07.07.01.00,Despesas Pré-Operacionais ou Pré-Industriais,"Contas que registram os gastos de organização e administração, encargos financeiros líquidos, estudos, projetos e detalhamentos, juros a acionista na fase de implantação e gastos preliminares de operação. O saldo existente em 31 de dezembro de 2008 no ativo diferido que, pela sua natureza, não puder ser alocado a outro grupo de contas, poderá permanecer no ativo sob essa classificação até sua completa amortização, sujeito à análise sobre a recuperação",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107070200,1.07.07.02.00,Despesas com Pesquisas Científicas ou Tecnológicas,"Contas que registram os gastos com pesquisa científica ou tecnológica. O saldo existente em 31 de dezembro de 2008 no ativo diferido que, pela sua natureza, não puder ser alocado a outro grupo de contas, poderá permanecer no ativo sob essa classificação até sua completa amortização, sujeito à análise sobre a recuperação",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107070300,1.07.07.03.00,Demais Aplicações em Despesas Amortizáveis,"Contas que registram os gastos com pesquisas e desenvolvimento de produtos, com a implantação de sistemas e métodos e com reorganização. O saldo existente em 31 de dezembro de 2008 no ativo diferido que, pela sua natureza, não puder ser alocado a outro grupo de contas, poderá permanecer no ativo sob essa classificação até sua completa amortização, sujeito à análise sobre a recuperação",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107070400,1.07.07.04.00,Correção Monetária - Diferença IPC/BTNF (Lei no 8.200/1991),"Contas/subcontas do ativo diferido que registram a correção monetária relativa à diferença, em relação ao ano de 1990, entre o IPC e o BTN Fiscal, na forma estabelecida nos arts. 32 e 33 do Decreto no 332, de 1991.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107070500,1.07.07.05.00,Correção Monetária Especial (Lei no 8.200/1991),"Contas/subcontas do ativo diferido que registram a correção monetária especial, na forma do art. 44 do Decreto no 332, de 1991.",account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_107070600,1.07.07.06.00,(-) Amortização do Diferido,Contas correspondentes à amortização das contas do ativo diferido.,account.data_account_type_current_assets,,l10n_br_account_chart_template
account_template_201010100,2.01.01.01.00,Fornecedores CIRCULANTE,"Contas que registram o valor a pagar correspondentes à compra de matérias-primas, bens, insumos e mercadorias.(Podem ser informados, também, os adiantamentos de clientes efetuados até 31.12.2008)",account.data_account_type_payable,TRUE,l10n_br_account_chart_template
account_template_201010101,2.01.01.01.01,Adiantamentos de Clientes,Contas que registram o valor correspondente a adiantamentos de clientes.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010201,2.01.01.02.01,Financiamentos a Curto Prazo - Sistema Financeiro Nacional,"Contas que registram os credores por financiamentos  a curto prazo, obtidos junto ao Sistema Financeiro Nacional, encargos financeiros a transcorrer e juros a pagar de empréstimos e financiamentos.  Atenção: as obrigações resultantes de operações de Arrendamento Mercantil (Leasing Financeiro) devem ser informadas na conta Financiamentos a Curto Prazo – Outros. ",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010202,2.01.01.02.02,Arrendamento Mercantil (Financeiro) a Curto Prazo - Sistema Financeiro Nacional,Contas que registram as obrigações de curto prazo relativas a arrendamento mercantil financeiro contratado junto a empresas integrantes do Sistema Financeiro Nacional,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010203,2.01.01.02.03,Financiamentos a Curto Prazo - Outros,"Contas que registram os credores por financiamentos a curto prazo, obtidos no Brasil, encargos financeiros a transcorrer e juros a pagar de empréstimos e financiamentos.  Atenção: as obrigações resultantes de financiamentos obtidos com pessoas físicas ou outras empresas que não sejam instituições financeiras devem ser informadas nesta conta .",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010204,2.01.01.02.04,Financiamentos a Curto Prazo - Exterior,"Contas que registram os credores por financiamentos a curto prazo, obtidos no exterior, encargos financeiros a transcorrer e juros a pagar de empréstimos e financiamentos.  Atenção: as obrigações resultantes de operações de Arrendamento Mercantil (Leasing Financeiro) contratadas no exterior devem ser informadas na conta Arrendamento Mercantil (Financeiro) a Curto Prazo – Exterior",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010205,2.01.01.02.05,Arrendamento Mercantil (Financeiro) a Curto Prazo - Exterior,Contas que registram as obrigações das pessoas jurídicas relativas a arrendamento mercantil financeiro contratado junto a empresas não sediadas no Brasil,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010301,2.01.01.03.01,IPI a Recolher,Contas correspondentes ao IPI a Recolher no final do período de apuração.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010302,2.01.01.03.02,ICMS e Contribuições a Recolher,Contas correspondentes ao ICMS a Recolher no final do período de apuração.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010303,2.01.01.03.03,Tributos Municipais a Recolher,Contas correspondentes a tributos municipais a Recolher no final do período de apuração.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010400,2.01.01.04.00,FGTS a Recolher,Contas que registram o valor do FGTS a recolher,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010500,2.01.01.05.00,PIS e COFINS a Recolher,Contas que registram o valor do PIS e da COFINS a recolher,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010600,2.01.01.06.00,Contribuições Previdenciárias a Recolher,Contas que registram  o valor das Contribuições Previdenciárias a recolher,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010690,2.01.01.06.90,Outros tributos a recolher,Contas correspondentes a tributos a recolher não classificáveis em contas específicas.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010700,2.01.01.07.00,Salários a Pagar,"Contas que registram o valor correspondente aos salários, ordenados, horas extras, adicionais e prêmios a serem pagos no exercício subseqüente.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010800,2.01.01.08.00,Dividendos Propostos ou Lucros Creditados,"Contas correspondentes aos dividendos aprovados pela Assembléia, creditados aos acionistas ou propostos pela administração da pessoa jurídica na data do balanço, como parte da destinação proposta para os lucros.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201010900,2.01.01.09.00,Provisão para a Contribuição Social sobre o Lucro Líquido,Conta correspondente à provisão para a contribuição social sobre o lucro líquido a pagar.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201011000,2.01.01.10.00,Provisão para o Imposto de Renda,Conta correspondente ao saldo a pagar da provisão para o imposto de renda.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201011100,2.01.01.11.00,Débitos Fiscais CSLL – Diferenças Temporárias,"As companhias abertas, obrigatoriamente, deverão informar, nestas contas, o valor dos débitos fiscais com realização no exercício seguinte e das diferenças temporárias, relativos à CSLL, conforme Deliberação CVM no 273, de 20 de agosto de 1998.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201011200,2.01.01.12.00,Débitos Fiscais IRPJ – Diferenças Temporárias,"As companhias abertas, obrigatoriamente, deverão informar, nestas contas, o valor dos débitos fiscais com realização no exercício seguinte e das diferenças temporárias, relativos ao IRPJ, conforme Deliberação CVM no 273, de 20 de agosto de 1998.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201011210,2.01.01.12.10,Provisões de Natureza Fiscal,"Contas que registram, a partir de 01.01.2008, outras provisões de natureza fiscal.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201011220,2.01.01.12.20,Provisões de Natureza Trabalhista,"Contas que registram, a partir de 01.01.2008, outras provisões de natureza trabalhista.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201011230,2.01.01.12.30,Provisões de Natureza Cível,"Contas que registram, a partir de 01.01.2008, outras provisões de natureza cível.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201011240,2.01.01.12.40,Doações e Subvenções para Investimentos,"Contas que registram, a partir de 01.01.2008, as doações e subvenções para investimento, enquanto não transferidas para o resultado do exercício.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201011300,2.01.01.13.00,Outras Contas,"Contas que registram comissões a pagar ou provisionadas de retenções contratuais, de obrigações decorrentes do fornecimento ou utilização de serviços  (energia elétrica, água, telefone, propaganda, honorários profissionais de terceiros, aluguéis) e outras contas não citadas nas contas anteriores. Atenção: também são incluídas, nesta conta, as provisões para registro de obrigações, tais como as provisões para: férias, gratificações a empregados (inclusive encargos sociais a pagar e FGTS a recolher sobre tais provisões), e outras de natureza semelhante, ainda que não dedutíveis.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_201019000,2.01.01.90.00,(-) Contas Retificadoras,Contas correspondentes às contas retificadoras do account.data_account_type_current_liabilities circulante.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010100,2.03.01.01.00,Fornecedores NÃO-CIRCULANTE,"Contas que registram valores a pagar relativos à compra de matérias-primas, bens, insumos e mercadorias e o valor correspondente a adiantamentos de clientes, com prazo de pagamento posterior ao exercício seguinte à data do balanço.",account.data_account_type_payable,TRUE,l10n_br_account_chart_template
account_template_203010201,2.03.01.02.01,Financiamentos a Longo Prazo - Sistema Financeiro Nacional,"Contas que registram os credores por financiamentos a longo prazo, obtidos junto ao Sistema Financeiro Nacional, encargos financeiros a transcorrer e juros a pagar de empréstimos e financiamentos. Atenção: as obrigações resultantes de operações de Arrendamento Mercantil (Leasing Financeiro) devem ser informadas na conta Financiamentos a Longo Prazo – Brasil – Outros ",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010202,2.03.01.02.02,Arrendamento Mercantil (Financeiro) a Longo Prazo - Sistema Financeiro Nacional,Contas que registram as obrigações de longo prazo relativas a arrendamento mercantil financeiro contratado junto a empresas integrantes do Sistema Financeiro Nacional,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010203,2.03.01.02.03,Financiamentos a Longo Prazo – Brasil - Outros,"Contas que registram os credores por financiamentos de longo prazo, obtidos no Brasil, encargos financeiros a transcorrer e juros a pagar de empréstimos e financiamentos.  Atenção: as obrigações resultantes de financiamentos obtidos com pessoas físicas ou outras empresas que não sejam instituições financeiras devem ser informadas nesta conta.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010204,2.03.01.02.04,Financiamentos a Longo Prazo – Exterior,"Contas que registram os credores por financiamentos a longo prazo, obtidos no exterior, encargos financeiros a transcorrer e juros a pagar de empréstimos e financiamentos.  Atenção: as obrigações resultantes de operações de Arrendamento Mercantil (Leasing Financeiro) contratadas no exterior devem ser informadas na conta Arrendamento Mercantil (Financeiro) a Longo Prazo – Exterior ",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010205,2.03.01.02.05,Arrendamento Mercantil (Financeiro) a Longo Prazo – Exterior,Contas que registram as obrigações de longo prazo relativas a arrendamento mercantil financeiro contratado junto a empresas não sediadas no Brasil,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010300,2.03.01.03.00,Empréstimos de Sócios/Acionistas Não Administradores,Contas relativas a empréstimos concedidos à pessoa jurídica por sócios e acionistas não administradores.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010400,2.03.01.04.00,Créditos de Pessoas Ligadas (Físicas/Jurídicas),"Contas que registram compras, adiantamentos ou empréstimos de sociedades coligadas ou controladas, diretores, acionistas ou participantes da empresa, que não constituam negócios usuais na exploração do objeto social da pessoa jurídica.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010500,2.03.01.05.00,Provisão para o Imposto de Renda sobre Lucros Diferidos,"Conta que registra o imposto de renda sobre lucros diferidos, tais como: lucro inflacionário não realizado, contratos a longo prazo relativos a fornecimento de bens e de construção por empreitada para o poder público e suas empresas, ganho de capital oriundo de desapropriação, ganho de capital por venda de bens do ativo permanente com recebimento parcelado a longo prazo e depreciação acelerada.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010600,2.03.01.06.00,Débitos Fiscais CSLL - Diferenças Temporárias,"As companhias abertas, obrigatoriamente, devem informar, nestas contas, o valor dos débitos fiscais com realização após o exercício seguinte e das diferenças temporárias, relativos à CSLL, conforme Deliberação CVM nº 273, de 1998",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010700,2.03.01.07.00,Débitos Fiscais IRPJ - Diferenças Temporárias,"As companhias abertas, obrigatoriamente, devem informar, nestas contas, o valor dos débitos fiscais com realização após o exercício seguinte e das diferenças temporárias, relativos ao IRPJ, conforme Deliberação CVM no 273, de 1998.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010710,2.03.01.07.10,Outras Provisões de Natureza Fiscal,"Contas que registram, a partir de 01.01.2008, as outras provisões de natureza fiscal, enquanto não transferidas para o resultado do exercício.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010720,2.03.01.07.20,Outras Provisões de Natureza Trabalhista,"Contas que registram, a partir de 01.01.2008, as outras provisões de natureza trabalhista, enquanto não transferidas para o resultado do exercício.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010730,2.03.01.07.30,Outras Provisões de Natureza Cível,"Contas que registram, a partir de 01.01.2008, as outras provisões de natureza cível, enquanto não transferidas para o resultado do exercício.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010740,2.03.01.07.40,Doações e Subvenções para Investimentos,"Contas que registram, a partir de 01.01.2008, as doações e subvenções para investimento, enquanto não transferidas para o resultado do exercício.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203010800,2.03.01.08.00,Outras Contas,"Contas que registram obrigações, não especificadas nos itens precedentes, cujo vencimento ocorrerá em período posterior ao exercício seguinte.  Atenção: não incluir, nesta conta, o valor contratado das vendas a prazo ou a prestação para recebimento após o término do ano-calendário subseqüente, no caso de atividade imobiliária, e os juros e demais receitas financeiras recebidos antecipadamente em transações financeiras. Esses valores devem ser informados em Resultados de Exercícios Futuros.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203019000,2.03.01.90.00,(-) Contas Retificadoras,Contas retificadoras do Exigível de Longo Prazo,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203030100,2.03.03.01.00,Receitas Diferidas,"Saldo remanescente da conta Resultado de Exercícios Futuros onde a pessoa jurídica que explore as atividades de compra e venda, loteamento, incorporação e construção de imóveis indicava o valor contratado das vendas a prazo ou a prestação para recebimento após o término do ano-calendário subseqüente, no caso de atividade imobiliária. Também se consideravam como receitas de exercícios futuros os juros e demais receitas financeiras recebidos antecipadamente em transações financeiras.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_203030300,2.03.03.03.00,(-) Custos Correspondentes às Receitas Diferidas,Contas correspondentes aos custos e despesas de exercícios futuros correspondentes às receitas indicadas na conta precedente.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207010100,2.07.01.01.00,Capital Subscrito de Domiciliados e Residentes no País,Contas correspondentes ao capital subscrito de domiciliados no País.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207010200,2.07.01.02.00,(-) Capital a Integralizar de Domiciliados e Residentes no País,Contas correspondentes ao capital social subscrito de domiciliados no País que não tenha sido integralizado.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207010300,2.07.01.03.00,Capital Subscrito de Domiciliados e Residentes no Exterior,Contas correspondentes ao capital subscrito de domiciliados no exterior.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207010400,2.07.01.04.00,(-) Capital a Integralizar de Domiciliados e Residentes no Exterior,Contas correspondentes ao capital social subscrito de domiciliados no exterior que não tenha sido integralizado.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207040100,2.07.04.01.00,Reservas de Capital,"Contas correspondentes às reservas constituídas pela correção monetária do capital, por incentivos fiscais, por ágio na emissão de ações, por alienação de partes beneficiárias.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207040200,2.07.04.02.00,Reservas de Reavaliação,"Contas correspondentes aos saldos dos reservas de reavaliação ainda não realizadas, decorrentes de reavaliação de ativos próprios e de ativos de coligadas e controladas, estes avaliados pelo método da equivalência patrimonial.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207040300,2.07.04.03.00,Reservas de Lucros,"Contas correspondentes às reservas constituídas pela destinação de lucros da empresa, tais como: reserva legal, reservas estatutárias, reserva para contingências, reserva de lucros a realizar, reserva de lucros para expansão, reserva especial para dividendo obrigatório não distribuído e reserva de exaustão incentivada de recursos minerais.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207040301,2.07.04.03.01,Reservas de Lucros - Doações e Subvenções para Investimentos,"Contas que registram, a partir de 01.01.2008, as doações e subvenções para investimento",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207040302,2.07.04.03.02,Reservas de Lucros - Prêmio na Emissão de Debêntures,"Contas que registram, a partir de 01.01.2008, os prêmios na emissão de debêntures",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207040400,2.07.04.04.00,"Reserva para Aumento de Capital (Lei no 9.249/1995, art. 9o, § 9o)","Conta correspondente à reserva constituída em 1996 com o montante dos juros sobre o capital próprio deduzidos como despesa financeira, mas mantidos no patrimônio da empresa, caso esta tenha optado pela faculdade prevista no § 9o do art. 9o da Lei no 9.249, de 1995.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207040500,2.07.04.05.00,Outras Reservas,"Contas correspondentes às demais reservas não consignadas nos itens anteriores, tais como o saldo devedor ou credor da conta de correção monetária correspondente à diferença, em relação ao ano de 1990, entre o IPC e o BTN Fiscal e o saldo da correção especial das contas do ativo permanente efetuada com base nos arts. 33 e 44 do Decreto no 332, de 1991.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207050100,2.07.05.01.00,Ajustes às Normas Internacionais de Contabilidade,"Contrapartidas de aumentos ou diminuições de valor atribuídos a elementos do ativo e do account.data_account_type_current_liabilities, em decorrência da sua avaliação a valor justo, nos casos previstos nesta Lei ou, em normas expedidas pela Comissão de Valores Mobiliários, com base na competência conferida pelo § 3o do art. 177 da Lei 6.404/76 (enquanto não computadas no resultado do exercício em obediência ao regime de competência,)
",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207050101,2.07.05.01.01,(-) Ajustes às Normas Internacionais de Contabilidade,"Contrapartidas de aumentos ou diminuições de valor atribuídos a elementos do ativo e do account.data_account_type_current_liabilities, em decorrência da sua avaliação a valor justo, nos casos previstos nesta Lei ou, em normas expedidas pela Comissão de Valores Mobiliários, com base na competência conferida pelo § 3o do art. 177 da Lei 6.404/76 (enquanto não computadas no resultado do exercício em obediência ao regime de competência,)
",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207070100,2.07.07.01.00,Lucros Acumulados e/ou Saldo à Disposição da Assembléia,Contas correspondentes aos lucros acumulados ou do saldo à disposição da assembléia.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207070200,2.07.07.02.00,(-) Prejuízos Acumulados,Contas correspondentes aos prejuízos acumulados.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207070300,2.07.07.03.00,(-) Ações em Tesouraria,Contas que registrem as aquisições de ações da própria empresa.,account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_207070400,2.07.07.04.00,Outras,"Outras contas classificáveis no patrimônio líquido que não tenham correspondência nas contas Lucros Acumulados e/ou Saldo à Disposição da Assembléia, Prejuízos Acumulados, Ações em Tesouraria.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_208010100,2.08.01.01.00,Fundo Patrimonial,"Contas que registrem, nas instituições imunes ou isentas, o Fundo Patrimonial.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_208040100,2.08.04.01.00,Reservas Patrimoniais,"Contas correspondentes, nas instituições imunes ou isentas, às reservas patrimoniais.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_208040200,2.08.04.02.00,Reservas Estatutárias,"Contas correspondentes, nas instituições imunes ou isentas, às reservas estatutárias.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_208070100,2.08.07.01.00,Superávits Acumulados,"Contas correspondentes, nas instituições imunes ou isentas, aos superávits acumulados.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_208070200,2.08.07.02.00,Déficits Acumulados,"Contas correspondentes, nas instituições imunes ou isentas, aos déficits acumulados.",account.data_account_type_current_liabilities,,l10n_br_account_chart_template
account_template_3010101010101,3.01.01.01.01.01.01,Receita de Exportação Direta de Mercadorias e Produtos,Contas que registram o valor da receita auferida em decorrência da exportação direta de mercadorias e produtos.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101010102,3.01.01.01.01.01.02,Receita de Vendas de Mercadorias e Produtos a Comercial Exportadora com Fim Específico de Exportação,"Contas que registram o valor da receita auferida em decorrência da venda de mercadorias e produtos a empresa comercial exportadora, com fim específico de exportação.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101010103,3.01.01.01.01.01.03,Receita de Exportação de Serviços,Contas que registram o valor da receita auferida em decorrência da exportação direta de serviços,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101010200,3.01.01.01.01.02.00,Receita da Venda no Mercado Interno de Produtos de Fabricação Própria,"Contas que registram a receita auferida no mercado interno correspondente à venda de produtos de fabricação própria e as receitas auferidas na industrialização por encomenda ou por conta e ordem de terceiros. (Não se incluem o valor correspondente ao Imposto sobre Produtos Industrializados (IPI) cobrado destacadamente do comprador ou contratante, uma vez que o vendedor é mero depositário e este imposto não integra o preço de venda da mercadoria, e, também, o valor correspondente ao ICMS cobrado na condição de substituto.)",account.data_account_type_revenue,,l10n_br_account_chart_template
account_template_3010101010300,3.01.01.01.01.03.00,Receita da Revenda de Mercadorias no Mercado Interno,"Contas que registram receita auferida no mercado interno, correspondente à revenda de mercadorias e o resultado auferido nas operações de conta alheia.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101010400,3.01.01.01.01.04.00,Receita da Prestação de Serviços – Mercado Interno,Contas que registram a receita decorrente dos serviços prestados.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101010500,3.01.01.01.01.05.00,Receita das Unidades Imobiliárias Vendidas,"As pessoas jurídicas que exploram atividades imobiliárias devem indicar, nestas contas, o montante das receitas das unidades imobiliárias vendidas, apropriadas ao resultado, inclusive as receitas transferidas de Resultados de Exercícios Futuros e os custos recuperados de períodos de apuração anteriores.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101010600,3.01.01.01.01.06.00,Receita de Locação de Bens Móveis e Imóveis,Contas que registram a receita decorrente da locação de bens móveis e imóveis,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101010700,3.01.01.01.01.07.00,Outras,Outras contas que registrem valores componentes da receita bruta não especificadas nos itens anteriores.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101030100,3.01.01.01.03.01.00,"(-) Vendas Canceladas, Devoluções e Descontos Incondicionais","Contas representativas das vendas canceladas, a devoluções de vendas e a descontos incondicionais concedidos sobre  receitas constantes das contas integrantes do grupo RECEITA BRUTA",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101030200,3.01.01.01.03.02.00,(-) ICMS,"Contas que registram o total do Imposto Sobre Operações Relativas à Circulação de Mercadorias e Sobre Prestação de Serviços de Transporte Interestadual e Intermunicipal e de Comunicação (ICMS) calculado sobre as receitas das vendas e de serviços constantes das contas integrantes do grupo RECEITA BRUTA. Informar o resultado da aplicação das alíquotas sobre as respectivas receitas, e não o montante recolhido, durante o período de apuração, pela pessoa jurídica.O valor referente ao ICMS pago como substituto não deve ser incluído nesta conta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101030300,3.01.01.01.03.03.00,(-) Cofins,"vigente à época da ocorrência dos fatos geradores, incidente sobre as receitas das contas integrantes do grupo RECEITA BRUTA. O valor informado deve ser apurado de forma centralizada pelo estabelecimento matriz, quando a pessoa jurídica possuir mais de um estabelecimento (Lei no 9.779, de 1999, art. 15, III). Não incluir a Cofins incidente sobre as demais receitas operacionais, que deverá ser informada em conta distinta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101030400,3.01.01.01.03.04.00,(-) PIS/Pasep,"Contas que registram as contribuições para o PIS/Pasep apurado sobre a receita de vendas em consonância com a legislação vigente à época da ocorrência dos fatos geradores, incidente sobre as receitas das contas integrantes do grupo RECEITA BRUTA. O valor informado deve ser apurado de forma centralizada pelo estabelecimento matriz, quando a pessoa jurídica possuir mais de um estabelecimento (Lei no 9.779, de 1999, art. 15, III). Não incluir o PIS/Pasep incidente sobre as demais receitas operacionais, que deverá ser informada em conta distinta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101030500,3.01.01.01.03.05.00,(-) ISS,"Contas que registram o Imposto sobre Serviço de qualquer Natureza (ISS) relativo às receitas de serviços,conforme legislação específica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010101030600,3.01.01.01.03.06.00,(-) Demais Impostos e Contribuições Incidentes sobre Vendas e Serviços,"Contas que registrem os demais impostos e contribuições incidentes sobre as receitas das vendas de que tratam as contas integrantes do grupo RECEITA BRUTA, que guardem proporcionalidade com o preço e sejam considerados redutores das receitas de vendas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010103010000,3.01.01.03.01.00.00,Custo dos Produtos de Fabricação Própria Vendidos, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010103030000,3.01.01.03.03.00.00,Custo das Mercadorias Revendidas, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010103050000,3.01.01.03.05.00.00,Custo dos Serviços Vendidos, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010103070100,3.01.01.03.07.01.00,Custo das Unidades Imobiliárias Vendidas,"Contas que registram, na empresa que tiver por objeto a compra de imóveis para venda ou que promover empreendimento de desmembramento ou loteamento de terrenos, incorporação imobiliária ou construção de prédio destinado à venda, os valores dos custos correspondentes às unidades imobiliárias vendidas apropriados ao resultado do período de apuração. A recuperação de custos do próprio período é computada no montante a ser indicado nesta conta. Os custos recuperados correspondentes a períodos de apuração anteriores devem ser indicados na conta Receita das Unidades Imobiliárias Vendidas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010100,3.01.01.05.01.01.00,Variações Cambiais Ativas,"Contas que registram os ganhos apurados em razão de variações ativas Decorrentes da atualização dos direitos de crédito e obrigações, calculados com base nas variações nas taxas de câmbio.
Atenção:
1) As variações cambiais ativas decorrentes dos direitos de crédito e de obrigações, em função da taxa de câmbio, são consideradas como receita financeira, inclusive para fins de cálculo do lucro da exploração (Lei nº 9.718, art. 9º c/c art. 17); 
2) Nas atividades de compra e venda, loteamento, incorporação e construção de imóveis, as variações cambiais ativas são reconhecidas como receita segundo as normas constantes da IN SRF nº 84/79, de 20 de dezembro de 1979, da IN SRF nº 23/83, de 25 de março de 1983, e da IN SRF nº 67/88, de 21 de abril de 1988 (IN SRF nº 25/99, de 25 de fevereiro de 1999).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010200,3.01.01.05.01.02.00,"Ganhos Auferidos no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório dos ganhos auferidos, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País;
b) os ganhos auferidos nas alienações, fora de bolsa, de ouro, ativo financeiro, e de participações societárias, exceto as alienações de participações societárias permanentes em sociedades coligadas e controladas e de participações societárias que permanecerem no ativo da pessoa jurídica até o término do ano-calendário seguinte ao de suas aquisições; e 
c) os rendimentos auferidos em operações de swap e no resgate de quota de fundo de investimento cujas carteiras sejam constituídas, no mínimo, por 67% (sessenta e sete por cento) de ações no mercado à vista de bolsa de valores ou entidade assemelhada (Lei nº 9.532, de 1997, art. 28, alterado pela MP nº 1.636, de 1998, art. 2º, e reedições).Considera-se ganho o resultado positivo auferido nas operações citadas acima, realizadas em cada mês, admitida a dedução dos custos e despesas incorridos, necessários à realização das operações.
Atenção:
1) Os ganhos auferidos em operações day-trade devem ser informados em conta específica.
2) O valor correspondente às perdas incorridas no mercado de renda variável, exceto day-trade, deve ser informado em conta específica.
3) São consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros as entidades cujo objeto social seja  análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010300,3.01.01.05.01.03.00,Ganhos em Operações Day-Trade,"Contas que registram os ganhos diários auferidos, em cada mês do período de apuração, em operações day-trade. Considera-se ganho o resultado positivo auferido nas operações citadas acima, realizadas em cada mês, admitida a dedução dos custos e despesas incorridos, necessários à realização das operações. Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado à vista, no mesmo dia. Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia. Atenção: o valor correspondente às perdas incorridas nas operações day-trade deve ser informado em conta específica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010400,3.01.01.05.01.04.00,Receitas de Juros sobre o Capital Próprio,"Contas que registram os juros recebidos, a título de remuneração do capital próprio, em conformidade com o art. 9o da Lei no 9.249, de 1995. O valor informado deve corresponder ao total dos juros recebidos antes do desconto do imposto de renda na fonte. O valor do imposto de renda retido na fonte, para as pessoas jurídicas tributadas pelo lucro real, é considerado antecipação do imposto devido no encerramento do período de apuração ou, ainda, pode ser compensado com aquele que for retido, pela beneficiária, por ocasião do pagamento ou crédito de juros a título de remuneração do capital próprio, ao seu titular ou aos seus sócios.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010500,3.01.01.05.01.05.00,Outras Receitas Financeiras,"Contas que registram receitas auferidas no período de apuração relativas a juros, descontos, lucro na operação de reporte, prêmio de resgate de títulos ou debêntures e rendimento nominal auferido em aplicações financeiras de renda fixa, não incluídas nas contas precedentes deste  grupo. As receitas dessa natureza, derivadas de operações com títulos vencíveis após o encerramento do período de apuração, serão rateadas segundo o regime de competência.Atenção:  1) As variações monetárias ativas decorrentes da atualização dos direitos de crédito e das obrigações, em função de índices ou coeficientes aplicáveis por disposição legal ou contratual, devem ser informadas como receita financeira;  2) As variações cambiais ativas devem ser informadas na conta Variações Cambiais Ativas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010600,3.01.01.05.01.06.00,Ganhos na Alienação de Participações Não Integrantes do Ativo Permanente,"Contas que registram os ganhos auferidos na alienação de ações, títulos ou quotas de capital não integrantes do ativo permanente, desde que não incluídos na conta Ganhos Auferidos no Mercado de Renda Variável, exceto Day-Trade.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010700,3.01.01.05.01.07.00,Resultados Positivos em Participações Societárias,"Contas que registram:
a) os lucros e dividendos derivados de investimentos avaliados pelo custo de aquisição;
b) os ganhos por ajustes no valor de investimentos relevantes avaliados pelo método da equivalência patrimonial, decorrentes de lucros apurados nas controladas e coligadas. Atenção: considera-se controlada a filial, a agência, a sucursal, a dependência ou o escritório de representação no exterior, sempre que os respectivos ativos e account.data_account_type_current_liabilitiess não estejam incluídos na contabilidade da investidora, por força de normatização específica.

c) as bonificações recebidas. Atenção: 1) as bonificações recebidas, decorrentes da incorporação de lucros ou reservas não tributados na forma do art. 35 da Lei nº 7.713, de 1988, ou apurados nos anos-calendário de 1994 ou 1995, são consideradas a custo zero, não afetando o valor do investimento nem o resultado do período de apuração (art. 3º da Lei nº 8.849, de 1994, e art. 3º da Lei nº 9.064, de 1995). 2) o caso de investimento avaliado pelo custo de aquisição, as bonificações recebidas, decorrentes da incorporação de lucros ou reservas tributados na forma do art. 35 da Lei nº 7.713, de 1988, e de lucros ou reservas apurados no ano-calendário de 1993 ou a partir do ano-calendário de 1996, são registradas tomando-se como custo o valor da parcela dos lucros ou reservas capitalizados.
d) os lucros e dividendos de participações societárias avaliadas pelo custo de aquisição; Atenção: os lucros ou dividendos recebidos em decorrência de participações societárias avaliadas pelo custo de aquisição adquiridas até 6 (seis) meses antes da data do recebimento devem ser registrados como diminuição do valor do custo, não sendo incluídos nesta conta. 
e) os resultados positivos decorrentes de participações societárias no exterior avaliadas pelo patrimônio líquido, os dividendos de participações avaliadas pelo custo de aquisição e os resultados de equivalência patrimonial relativos a filiais, sucursais ou agências da pessoa jurídica localizadas no exterior, em decorrência de operações realizadas naquelas filiais, sucursais ou agências. Os lucros auferidos no exterior serão adicionados ao lucro líquido, para efeito de determinação do lucro real, no período de apuração correspondente ao balanço levantado em 31 de dezembro do ano-calendário em que tiverem sido disponibilizados, observando-se o disposto nos arts. 394 e 395 do Decreto nº 3.000, de 1999, e no art. 74 da Medida Provisória nº 2.158-35, de 24 de agosto de 2001.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010710,3.01.01.05.01.07.10,Amortização de Deságio nas Aquisições de Investimentos Avaliados pelo Patrimônio Líquido,"Contas que registram as amortizações de deságios nas aquisições de investimentos avaliados pelo patrimônio líquido. O valor amortizado que for excluído do lucro líquido para determinação do lucro real deve ser controlado na Parte B do Livro de Apuração do Lucro Real até a alienação ou baixa da participação societária, quando, então, deve ser adicionado ao lucro líquido para determinação do lucro real no período de apuração em que for computado o ganho ou perda de capital havido.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010800,3.01.01.05.01.08.00,Resultados Positivos em SCP,"Conta utilizada pelas pessoas jurídicas que forem sócias ostensivas de sociedades em conta de participação, para o registro: 
a) de lucros derivados de participação em SCP, avaliadas pelo custo de aquisição;
b) dos ganhos por ajustes no valor de participação em SCP, avaliadas pelo método da equivalência patrimonial.
Atenção: os lucros recebidos de investimento em SCP, avaliado pelo custo de aquisição, ou a contrapartida do ajuste do investimento ao valor do patrimônio líquido da SCP, no caso de investimento avaliado por esse método, podem ser excluídos na determinação do lucro real dos sócios, pessoas jurídicas, das referidas sociedades (Decreto nº 3.000, de 1999, art. 149).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105010900,3.01.01.05.01.09.00,Rendimentos e Ganhos de Capital Auferidos no Exterior,"Contas que registram os rendimentos e ganhos de capital auferidos no exterior diretamente pela pessoa jurídica domiciliada no Brasil, pelos seus valores antes de descontado o tributo pago no país de origem. Esses valores podem, no caso de apuração trimestral do imposto, ser excluídos na apuração do lucro real do 1o ao 3o trimestres, devendo ser adicionados ao lucro líquido na apuração do lucro real referente ao 4º trimestre. Atenção: Os ganhos de capital referentes a alienações de bens e direitos do ativo permanente situados no exterior devem ser informados na conta Outras Receitas Não Operacionais..",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105011000,3.01.01.05.01.10.00,Reversão dos Saldos das Provisões Operacionais,"Contas que registram a reversão de saldos não utilizados das provisões constituídas no balanço do período de apuração imediatamente anterior para fins de apuração do lucro real (Lei no 9.430, de 1996, art. 14).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105011010,3.01.01.05.01.10.10,Prêmios Recebidos na Emissão de Debêntures,"Contas que registram, a partir de 01.01.2008, os prêmios recebidos na emissão de debêntures.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105011020,3.01.01.05.01.10.20,Doações e Subvenções para Investimentos,"Contas que registram, a partir de 01.01.2008, as doações e subvenções para investimento.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105011030,3.01.01.05.01.10.30,Contrapartida dos Ajustes ao Valor Presente,"Contrapartida do ajuste ao valor presente dos elementos do ativo e do account.data_account_type_current_liabilities (art. 183, inciso VIII, e art. 184, inciso III da Lei 6.404/76)",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105011040,3.01.01.05.01.10.40,Contrapartida de outros Ajustes às Normas Internacionais de Contabilidade,Contrapartida de outros ajustes decorrentes da adequação às Normas Internacionais de Contabilidade,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010105011100,3.01.01.05.01.11.00,Outras Receitas Operacionais,"Contas que registram todas as demais receitas que, por definição legal, sejam consideradas operacionais, tais como:
a) aluguéis de bens por empresa que não tenha por objeto a locação de móveis e imóveis;
b) recuperações de despesas operacionais de períodos de apuração anteriores, tais como: prêmios de seguros, importâncias levantadas das contas vinculadas do FGTS, ressarcimento de desfalques, roubos e furtos, etc. As recuperações de custos e despesas no decurso do próprio período de apuração devem ser creditadas diretamente às contas de resultado em que foram debitadas; 
c) os créditos presumidos do IPI para ressarcimento do valor da Contribuição ao PIS/Pasep e Cofins;
d) multas ou vantagens a título de indenização em virtude de rescisão contratual (Lei nº 9.430, de 1996, art. 70, § 3º, II);
e) o crédito presumido da contribuição para o PIS/Pasep e da Cofins concedido na forma do art. 3º da Lei nº 10.147, de 2000.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010100,3.01.01.07.01.01.00,Remuneração a Dirigentes e a Conselho de Administração,"Contas que registram a despesa incorrida relativa à remuneração mensal e fixa atribuída ao titular de firma individual, aos sócios, diretores e administradores de sociedades, ou aos representantes legais de sociedades estrangeiras, as despesas incorridas com os salários indiretos concedidos pela empresa a 
administradores, diretores, gerentes e seus assessores (PN Cosit nº 11, de 1992), e o valor referente às remunerações atribuídas aos membros do conselho fiscal/administração/consultivo.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010201,3.01.01.07.01.02.01,"Ordenados, Salários Gratificações e Outras Remunerações a Empregados","Contas que registram as despesas com ordenados, salários, gratificações e outras despesas com empregados, tais como: comissões, moradia, seguro de vida e outras de caráter remuneratório.
Atenção:
1) As despesas correspondentes a salários, ordenados, gratificações e outras remunerações referentes à área de saúde, tais como assistência médica, odontológica e farmacêutica, devem ser indicadas na conta específica.
2) Não deve ser informado nesta conta o valor referente às participações dos empregados no lucro da pessoa jurídica. Esse valor deve ser informado na conta Participações de Empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010203,3.01.01.07.01.02.03,Planos de Poupança e Investimentos de Empregados,Contas que registram o valor total dos gastos efetuados com Planos de Poupança e Investimentos (PAIT).,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010205,3.01.01.07.01.02.05,Fundo de Aposentadoria Programada Individual de Empregados,Contas que registram o valor total dos gastos efetuados com Fundos de Aposentadoria Programada Individual (FAPI).,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010207,3.01.01.07.01.02.07,Plano de Previdência Privada de Empregados,Contas que registram o valor total dos gastos efetuados com Planos de Previdência Privada.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010209,3.01.01.07.01.02.09,Outros Gastos com Pessoal,"Contas que registram os gastos com empregados não enquadrados nas contas precedentes
Atenção: 
1) As despesas correspondentes a salários, ordenados, gratificações e outras remunerações referentes à área de saúde, tais como assistência médica, odontológica e farmacêutica, devem ser indicadas na conta  Assistência Médica, Odontológica e Farmacêutica a Empregados;
2) não deve ser informado nesta conta o valor referente às participações dos empregados no lucro da pessoa jurídica. Esse valor deve ser informado na conta Participações de Empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010300,3.01.01.07.01.03.00,Prestação de Serviços por Pessoa Física sem Vínculo Empregatício,"Contas que registram, salvo se houver conta mais específica, as despesas correspondentes aos serviços prestados por pessoa física que não tenha vínculo empregatício com a pessoa jurídica, tais como: comissões, corretagens, gratificações, honorários, direitos autorais e outras remunerações, inclusive as relativas a empreitadas de obras exclusivamente de trabalho e as decorrentes de fretes e carretos em geral.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010400,3.01.01.07.01.04.00,Prestação de Serviço Pessoa Jurídica,"Contas que registram, salvo se houver conta mais específica, o valor das despesas correspondentes aos serviços prestados por outra pessoa jurídica à pessoa jurídica declarante.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010401,3.01.01.07.01.04.01,Serviços Prestados por Cooperativa de Trabalho,Contas que registram os serviços prestados por cooperativa de trabalho,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010402,3.01.01.07.01.04.02,Locação de Mão-de-obra,"Contas que registram o valor total dos gastos efetuados no período com a contratação de serviços executados mediante cessão de mão-de-obra ou empreitada, inclusive em regime temporário, sujeitos à retenção de contribuição previdenciária, nos termos do art. 219 do Regulamento da Previdência Social - RPS, aprovado pelo Decreto nº 3.048, de 1999",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010500,3.01.01.07.01.05.00,Encargos Sociais – Previdência Social,"Contas que registram as contribuições para a Previdência Social, não computadas nos custos (inclusive dos dirigentes – PN CST no 35, de 31 de agosto de 1981).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010600,3.01.01.07.01.06.00,Encargos Sociais – FGTS,"Contas que registram as contribuições para a o FGTS, não computadas nos custos (inclusive dos dirigentes - PN CST no 35, de 31 de agosto de 1981).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010700,3.01.01.07.01.07.00,Encargos Sociais – Outros,"Contas que registram os demais encargos sociais, não computados nos custos ou nas contas Encargos Sociais - Previdência Social ou Encargos Sociais - FGTS",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010800,3.01.01.07.01.08.00,Doações e Patrocínios de Caráter Cultural e Artístico (Lei no 8.313/1991),"Contas que registram as doações e patrocínios efetuados no período de apuração em favor de projetos culturais previamente aprovados pelo Ministério da Cultura ou pela Agência Nacional do Cinema (Ancine), observada a legislação de concessão dos projetos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107010900,3.01.01.07.01.09.00,"Doações a Instituições de Ensino e Pesquisa (Lei nº 9.249/1995, art.13, § 2º)","Contas que registram as doações a instituições de ensino e pesquisa cuja criação tenha sido autorizada por lei federal e que preencham os requisitos dos incisos I e II do art. 213 da Constituição Federal, de 1988, que são:
a) comprovação de finalidade não-lucrativa e aplicação dos excedentes financeiros em educação;
b) assegurar a destinação do seu patrimônio a outra escola comunitária, filantrópica ou confessional, ou ao Poder Público, no caso de encerramento de suas atividades.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011000,3.01.01.07.01.10.00,Doações a Entidades Civis,"Contas que registram as doações efetuadas a:
a) entidades civis, legalmente constituídas no Brasil, sem fins lucrativos, que prestem serviços gratuitos em benefício de empregados da pessoa jurídica doadora, e respectivos dependentes, ou em benefício da comunidade na qual atuem; e
b) Organizações da Sociedade Civil de Interesse Público (OSCIP), qualificadas segundo as normas estabelecidas na Lei no 9.790, de 23 de março de 1999.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011100,3.01.01.07.01.11.00,Outras Contribuições e Doações,"Contas que registram as doações feitas, entre outras, aos Fundos controlados pelos Conselhos Municipais, Estaduais e Nacional dos Direitos da Criança e do Adolescente.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011200,3.01.01.07.01.12.00,Alimentação do Trabalhador,"Contas que registram as despesas com alimentação do pessoal não ligado à produção, realizadas durante o período de apuração, ainda que a pessoa jurídica não tenha Programa de Alimentação do Trabalhador aprovado pelo Ministério do Trabalho.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011300,3.01.01.07.01.13.00,PIS/Pasep,Contas que registram as Contribuições para o PIS/Pasep incidente sobre as demais receitas operacionais.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011400,3.01.01.07.01.14.00,Cofins,Contas que registram a parcela da Cofins incidente sobre as demais receitas operacionais.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011500,3.01.01.07.01.15.00,CPMF,Contas que registram a Contribuição Provisória sobre Movimentação ou Transmissão de Valores e de Créditos de Natureza Financeira.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011600,3.01.01.07.01.16.00,"Demais Impostos, Taxas e Contribuições, exceto IR e CSLL","Contas que registram os demais Impostos, Taxas e Contribuições, exceto: 
a) incorporadas ao custo de bens do ativo permanente;
b) correspondentes aos impostos não recuperáveis, incorporados ao custo das matérias-primas, materiais secundários, materiais de embalagem e mercadorias destinadas à revenda;
c) correspondentes aos impostos recuperáveis;
d) correspondentes aos impostos e contribuições redutores da receita bruta;
e) correspondentes às Contribuições para o PIS/Pasep e à Cofins incidentes sobre as demais receitas operacionais, e à CPMF, indicados em contas específicas;
f) correspondentes à contribuição social sobre o lucro líquido e ao imposto de renda devidos, que são informados em contas específicas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011700,3.01.01.07.01.17.00,Arrendamento Mercantil,"Contas que registram as despesas, não computadas nos custos, pagas ou creditadas a título de contraprestação de arrendamento mercantil, decorrentes de contrato celebrado com observância da Lei no 6.099, de 12 de setembro de 1974, com as alterações da Lei no 7.132, de 26 de outubro de 1983, e da Portaria MF no 140, de 1984",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011800,3.01.01.07.01.18.00,Aluguéis,Contas que registram as despesas com aluguéis não decorrentes de arrendamento mercantil.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107011900,3.01.01.07.01.19.00,Despesas com Veículos e de Conservação de Bens e Instalações,"Contas que registram as despesas relativas aos bens que não estejam ligados diretamente à produção, as realizadas com reparos que não impliquem aumento superior a um ano da vida útil do bem, prevista no ato de sua aquisição, e as relativas a combustíveis e lubrificantes para veículos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012001,3.01.01.07.01.20.01,"Propaganda, Publicidade e Patrocínio (Associações Desportivas que Mantenham Equipe de Futebol Profissional)","Contas que registram as despesas relativas a propaganda publicidade e patrocínio com associações desportivas que mantenham equipe de futebol profissional e possuam registro na Federação de Futebol do respectivo Estado, a título de propaganda, publicidade e patrocínio.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012002,3.01.01.07.01.20.02,"Propaganda, Publicidade e Patrocínio","Contas que registram de propaganda, publicidade, exceto as classificadas na conta precedente",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012100,3.01.01.07.01.21.00,Multas,Contas que registram as despesas com multas.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012200,3.01.01.07.01.22.00,Encargos de Depreciação e Amortização,"Contas que registram apenas os encargos a esses títulos, com bens não aplicados diretamente na produção. Inclui a amortização dos ajustes de variação cambial contabilizada no ativo diferido, relativa à atividade geral da pessoa jurídica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012300,3.01.01.07.01.23.00,Perdas em Operações de Crédito,Contas que registram as perdas no recebimento de créditos decorrentes das atividades da pessoa jurídica.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012400,3.01.01.07.01.24.00,Provisões para Férias e 13o Salário de Empregados,"Contas que registram as despesas com a constituição de provisões para:
a) pagamento de remuneração correspondente a férias e adicional de férias de empregados, inclusive encargos sociais (Decreto no 3.000, de 1999, art. 337, e PN CST no 7, de 1980); 
b) o 13o salário, no caso de apuração trimestral do imposto, inclusive encargos sociais (Decreto no 3.000, de 1999, art. 338).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012500,3.01.01.07.01.25.00,Provisão para Perda de Estoque,Contas que registram as despesas com a constituição de provisão para perda de estoque,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012600,3.01.01.07.01.26.00,Demais Provisões,Contas que registram as despesas com provisões não relacionadas  em contas específicas,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012700,3.01.01.07.01.27.00,Gratificações a Administradores,Contas que registram as gratificações a administradores.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012800,3.01.01.07.01.28.00,Royalties e Assistência Técnica – PAÍS,"Contas que registram as despesas correspondentes às importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no Brasil, a título de royalties e assistência técnica, científica ou assemelhada, que não estejam relacionados com a produção de bens e/ou serviços.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107012900,3.01.01.07.01.29.00,Royalties e Assistência Técnica – EXTERIOR,"Contas que registram as despesas correspondentes às importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no exterior, a título de royalties e assistência técnica, científica ou assemelhada, que não estejam relacionados com a produção de bens e/ou serviços.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107013000,3.01.01.07.01.30.00,"Assistência Médica, Odontológica e Farmacêutica a Empregados","Indicar o valor das despesas com assistência médica, odontológica e farmacêutica. 
Atenção: o valor referente à contratação de serviços de profissionais liberais sem vínculo empregatício ou de sociedades civis deve ser informado nas contas Prestação de Serviços por Pessoa Física sem Vínculo Empregatício ou Prestação de Serviço Pessoa Jurídica, conforme o caso.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107013100,3.01.01.07.01.31.00,Pesquisas Científicas e Tecnológicas,"Contas que registram as despesas efetuadas a esse título, inclusive a contrapartida das amortizações daquelas registradas no ativo diferido",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107013200,3.01.01.07.01.32.00,Bens de Natureza Permanente Deduzidos como Despesa,"Contas que registram as despesas com aquisição de bens do ativo imobilizado cujo prazo de vida útil não ultrapasse um ano, ou, caso exceda esse prazo, tenha valor unitário igual ou inferior ao fixado no art. 301 do Decreto no 3.000, de 1999.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107013301,3.01.01.07.01.33.01,"Despesas com viagens, diárias e ajusta de custo","Contas que registram as despesas operacionais com viagens, diárias e ajuda de custo",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010107013390,3.01.01.07.01.33.90,Outras Despesas Operacionais,"Contas que registram as demais despesas operacionais, cujos títulos não se adaptem à nomenclatura específica desta ficha, tais como:
a) contribuição sindical;
b) prêmios de seguro;
c) fretes e carretos que não componham os custos;
d) transporte de empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010100,3.01.01.09.01.01.00,(-) Variações Cambiais Passivas,"Contas que registram as perdas monetárias passivas resultantes da atualização dos direitos de créditos e das obrigações, calculadas com base nas variações nas taxas de câmbio (Lei no 9.069, de 1995, art.52, e Lei no 9.249, de 1995, art. 8o).Inclusive a variação cambial passiva correspondente: 
a) à atualização das obrigações e dos créditos em moeda estrangeira, registrada em qualquer data e apurada no encerramento do período de apuração em função da taxa de câmbio vigente;
b) às operações com moeda estrangeira e conversão de obrigações para moeda nacional, ou novação dessas obrigações, ou sua extinção, total ou parcial, em virtude de capitalização, dação em pagamento, compensação, ou qualquer outro modo, desde que observadas as condições fixadas pelo Banco Central do Brasil.
Atenção: a amortização dos ajustes de variação cambial contabilizada no ativo diferido deve ser informada na conta Encargos de Depreciação e Amortização (Lei no 9.816, de 1999, art. 2o, e Lei no 10.305, de 2001).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010200,3.01.01.09.01.02.00,"(-) Perdas Incorridas no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório das perdas incorridas, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País;
b) as perdas incorridas nas alienações, fora de bolsa, de ouro, ativo financeiro, e de participações societárias, exceto as alienações de participações societárias permanentes em sociedades coligadas e controladas e de participações societárias que permanecerem no ativo da pessoa jurídica até o término do ano-calendário seguinte ao de suas aquisições; e
c) as perdas em operações de swap e no resgate de quota de fundo de investimento que mantenha, no mínimo, 67% (sessenta e sete por cento) de ações negociadas no mercado à vista de bolsa de valores ou entidade assemelhada (Lei no 9.532, de 1997, art. 28, alterado pela MP no 1.636, de 1998, art. 2o, e reedições). São consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros as entidades cujo objeto social seja análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM).
Atenção: as perdas apuradas em operações day-trade devem ser informadas em conta própria.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010300,3.01.01.09.01.03.00,(-) Perdas em Operações Day-Trade,"Contas que registram o somatório das perdas diárias apuradas, em cada mês do período de apuração, em operações day-trade.Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado à vista, no mesmo dia.Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010400,3.01.01.09.01.04.00,(-) Juros sobre o Capital Próprio,"Contas que registram as despesas com juros pagos ou creditados individualizadamente a titular, sócios ou acionistas, a título de remuneração do capital próprio, calculados sobre as contas do patrimônio liquido e limitados à variação, pro rata dia, da Taxa de Juros de Longo Prazo (TJLP) observando-se o regime de competência (Lei no 9.249, de 1995, art. 9o).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010500,3.01.01.09.01.05.00,(-) Outras Despesas Financeiras,"Contas que registram as despesas relativas a juros, não incluídas nas em outras contas, a descontos de títulos de crédito e ao deságio na colocação de debêntures ou outros títulos. Tais despesas serão obrigatoriamente rateadas, segundo o regime de competência.
Atenção:
1) as variações monetárias passivas decorrentes da atualização das obrigações, em função de índices ou coeficientes aplicáveis por disposição legal ou contratual, devem ser informadas como despesa financeira;
2) as variações cambiais passivas não devem ser informadas nesta conta, e sim na conta Variações Cambiais Passivas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010600,3.01.01.09.01.06.00,(-) Prejuízos na Alienação de Participações Não Integrantes do Ativo Permanente,"Contas que registram os prejuízos havidos em virtude de alienação de ações, títulos ou quotas de capital não integrantes do ativo permanente, desde que não incluídos nas contas Perdas Incorridas no Mercado de Renda Variável, exceto Day-Trade ou Perdas em Operações Day-Trade.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010700,3.01.01.09.01.07.00,(-) Resultados Negativos em Participações Societárias,"Contas que registram as perdas por ajustes no valor de investimentos relevantes avaliados pelo método da equivalência patrimonial, decorrentes de prejuízos apurados nas controladas e coligadas. 
Atenção:considera-se controlada a filial, a agência, a sucursal, a dependência ou o escritório de representação no exterior, sempre que os respectivos ativos e account.data_account_type_current_liabilitiess não estejam incluídos na contabilidade da investidora, por força de normatização específica. Devem, também, ser indicados nesta conta os resultados negativos derivados de participações societárias no exterior, avaliadas pelo patrimônio líquido. Incluem-se, nestas informações, as perdas apuradas em filiais, sucursais e agências da pessoa jurídica localizadas no exterior.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010710,3.01.01.09.01.07.10,(-) Amortização de Ágio nas Aquisições de Investimentos Avaliados pelo Patrimônio Líquido,"Contas que registram o valor da amortização registrada no período, referente ao ágio nas aquisições de investimentos avaliados pelo método da equivalência patrimonial.
Atenção: O valor amortizado deve ser adicionado ao lucro líquido, para determinação do lucro real, e controlado na Parte B do Livro de Apuração do Lucro Real até a alienação ou baixa da participação societária, quando, então, pode ser excluído do lucro líquido, para determinação do lucro real.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010800,3.01.01.09.01.08.00,(-) Resultados Negativos em SCP,"Conta utilizada pelos sócios ostensivos, pessoas jurídicas, de sociedades em conta de participação, para indicar as perdas por ajustes no valor de participação em SCP, avaliada pelo método da equivalência patrimonial.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109010900,3.01.01.09.01.09.00,(-) Perdas em Operações Realizadas no Exterior,"Contas que registram as perdas em operações realizadas no exterior diretamente pela pessoa jurídica domiciliada no Brasil, com exceção das perdas de capital decorrentes da alienação de bens e direitos do ativo permanente situados no exterior, que devem ser indicadas na conta Outras Despesas Não Operacionais.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109011000,3.01.01.09.01.10.00,(-) Contrapartida dos Ajustes ao Valor Presente,"Contrapartida do ajuste ao valor presente dos elementos do ativo e do account.data_account_type_current_liabilities (art. 183, inciso VIII, e art. 184, inciso III da Lei 6.404/76)",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109011100,3.01.01.09.01.11.00,(-) Contrapartida de outros Ajustes às Normas Internacionais de Contabilidade,Contrapartida de outros ajustes decorrentes da adequação às Normas Internacionais de Contabilidade,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010109011200,3.01.01.09.01.12.00,(-) Contrapartida dos Ajustes de Valor do Imobilizado e Intangível,"Contrapartida dos ajustes decorrentes da análise de recuperação dos valores registrados no imobilizado e no  intangível (art. 183, § 3º, da Lei 6.404/76)",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010301010100,3.01.03.01.01.01.00,Receitas de Alienações de Bens e Direitos do Ativo Permanente,"Contas que registram as receitas auferidas por meio de alienações, inclusive por desapropriação, de bens e direitos do ativo permanente. O valor relativo às receitas obtidas pela venda de sucata e de bens ou direitos do ativo permanente baixados em virtude de terem se tornado imprestáveis, obsoletos ou caído em desuso deve ser informado na conta Outras Receitas Não Operacionais Os valores correspondentes ao ganho ou perda de capital decorrente da alienação de bens e direitos do ativo permanente situados no exterior devem ser indicados, pelo seu resultado, nas contas Outras Receitas Não Operacionais ou Outras Despesas Não Operacionais, conforme o caso.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010301010110,3.01.03.01.01.01.10,Ganhos de Capital por Variação Percentual em Participação Societária Avaliada pelo Patrimônio Líquido,"Contas que registram o ganho de capital resultante de acréscimo, por variação percentual, do valor do patrimônio líquido de investimento avaliado pelo método da equivalência patrimonial.
Atenção: Esse valor deve ser excluído do lucro líquido para determinação do lucro real no período de apuração.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010301010200,3.01.03.01.01.02.00,Outras Receitas Não Operacionais,"Contas que registram:
a) todas as demais receitas decorrentes de operações não incluídas nas atividades principais e acessórias da empresa, tais como: a reversão do saldo da provisão para perdas prováveis na realização de investimentos e a reserva de reavaliação realizada no período de apuração, quando computada em conta de resultado; 
b) os ganhos de capital por variação na percentagem de participação no capital social de coligada ou controlada, quando o investimento for avaliado pela equivalência patrimonial (Decreto no 3.000, de 1999, art. 428);
c) os ganhos de capital decorrentes da alienação de bens e direitos do ativo permanente situados no exterior. Devem ser indicadas tanto as contas que registram as receitas quanto as que registram os custos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_301030103,3.01.03.01.03,DESPESAS NÃO OPERACIONAIS,,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010301030100,3.01.03.01.03.01.00,(-) Valor Contábil dos Bens e Direitos Alienados,"Contas que registram o contábil dos bens do ativo permanente baixados no curso do período de apuração cuja receita da venda tenha sido indicada na conta Receitas de Alienações de Bens e Direitos do Ativo Permanente.  O valor contábil de bens ou direitos baixados em virtude de terem se tornado imprestáveis, obsoletos ou caído em desuso e o valor contábil de bens ou direitos situados no exterior devem ser informados na conta Outras Despesas Não Operacionais.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010301030110,3.01.03.01.03.01.10,(-) Perdas de Capital por Variação Percentual em Participação Societária Avaliada pelo Patrimônio Líquido,"Contas que registram a perda de capital resultante de redução, por variação percentual, do valor do patrimônio líquido de investimento avaliado pelo método da equivalência patrimonial.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010301030200,3.01.03.01.03.02.00,(-) Outras Despesas Não Operacionais,"Contas que registram:
a) o valor contábil dos bens do ativo permanente baixados no curso do período de apuração não incluídos na conta precedente e a despesa com a constituição da provisão para perdas prováveis na realização de investimentos. 
Atenção: Sobre a definição de valor contábil, consultar o § 1o do art. 418 e o art. 426 do Decreto no 3.000, de 1999.
b) as perdas de capital por variação na percentagem de participação no capital social de coligada ou controlada no Brasil, quando o investimento for avaliado pela equivalência patrimonial (Decreto no 3.000, de 1999, art. 428).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010501010100,3.01.05.01.01.01.00,(-) Participações de Empregados,"Contas que registram as participações atribuídas a empregados segundo disposição legal, estatutária, contratual ou por deliberação da assembléia de acionistas ou sócios.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010501010200,3.01.05.01.01.02.00,(-) Contribuições para Assistência ou Previdência de Empregados,"Contas que registram as contribuições para instituições ou fundos de assistência ou previdência de empregados, baseadas nos lucros. Não indicar, nesta conta, aquelas contribuições já deduzidas como custo ou despesa operacional.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010501010300,3.01.05.01.01.03.00,(-) Outras Participações de Empregados,Contas que registram outras participações de empregados,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010501030100,3.01.05.01.03.01.00,(-) Participações de Administradores e Partes Beneficiárias,"Contas que registram quaisquer participações nos lucros atribuídas a administradores, sócio, titular de empresa individual e a portadores de partes beneficiárias, durante o período de apuração.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010501030200,3.01.05.01.03.02.00,(-) Participações de Debêntures ,Contas que representam as participações nos lucros da companhia atribuídas a debêntures de sua emissão,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3010501030300,3.01.05.01.03.03.00,(-) Outras ,Contas que registram outras participações,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3020101010100,3.02.01.01.01.01.00,(-) Contribuição Social sobre o Lucro Líquido,"Contas que registram as provisões para a CSLL calculadas sobre a base de cálculo correspondente ao período de apuração e sobre os lucros diferidos da atividade geral, se for o caso. A sua constituição é obrigatória para todas as pessoas jurídicas tributadas com base no lucro real. As cooperativas devem informar, nesta conta, a provisão da CSLL sobre os resultados das operações realizadas com os não-associados.
Atenção: para as empresas com atividades mistas, os valores da CSLL relativos às atividades em geral e atividade rural devem ser informados nas contas específicas de cada atividade (""Atividades em Geral"" e ""Atividade Rural"", respectivamente).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3020101010200,3.02.01.01.01.02.00,(-) Provisão para Imposto de Renda - Pessoa Jurídica,"Contas que registram as provisões para o IRPJ calculadas sobre a base de cálculo correspondente ao período de apuração e sobre os lucros diferidos da atividade geral, se for o caso. A sua constituição é obrigatória para todas as pessoas jurídicas tributadas com base no lucro real. As cooperativas devem informar, nesta conta, a provisão para o IRPJ sobre os resultados das operações realizadas com os não-associados.
Atenção: para as empresas com atividades mistas, os valores do IRPJ relativos às atividades em geral e atividade rural devem ser informados nas contas específicas de cada atividade (""Atividades em Geral"" e ""Atividade Rural"", respectivamente).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050101010100,3.05.01.01.01.01.00,Receita da Atividade Rural,Contas que registram a receita da atividade rural. ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050101030100,3.05.01.01.03.01.00,"(-) Vendas Canceladas, Devoluções e Descontos Incondicionais","Contas representativas das vendas canceladas, a devoluções de vendas e a descontos incondicionais concedidos sobre  receitas constantes da conta Receita da Atividade Rural.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050101030200,3.05.01.01.03.02.00,(-) ICMS,"Contas que registram o total do Imposto Sobre Operações Relativas à Circulação de Mercadorias e Sobre Prestação de Serviços de Transporte Interestadual e Intermunicipal e de Comunicação (ICMS) calculado sobre as receitas das vendas e de serviços constantes da conta Receita da Atividade Rural. Informar o resultado da aplicação das alíquotas sobre as respectivas receitas, e não o montante recolhido, durante o período de apuração, pela pessoa jurídica.O valor referente ao ICMS pago como substituto não deve ser incluído nesta conta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050101030300,3.05.01.01.03.03.00,(-) Cofins,"Contas que registram a Cofins apurada sobre a receita de vendas em consonância com a legislação vigente à época da ocorrência dos fatos geradores, incidente sobre as receitas da conta Receita da Atividade Rural. O valor informado deve ser apurado de forma centralizada pelo estabelecimento matriz, quando a pessoa jurídica possuir mais de um estabelecimento (Lei no 9.779, de 1999, art. 15, III).
 Não incluir a Cofins incidente sobre as demais receitas operacionais, que deverá ser informada em conta distinta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050101030400,3.05.01.01.03.04.00,(-) PIS/Pasep,"Contas que registram as contribuições para o PIS/Pasep apurado sobre a receita de vendas em consonância com a legislação vigente à época da ocorrência dos fatos geradores, incidente sobre as receitas da conta Receita da Atividade Rural. O valor informado deve ser apurado de forma centralizada pelo estabelecimento matriz, quando a pessoa jurídica possuir mais de um estabelecimento (Lei no 9.779, de 1999, art. 15, III). Não incluir o PIS/Pasep incidente sobre as demais receitas operacionais, que deverá ser informada em conta distinta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050101030500,3.05.01.01.03.05.00,(-) ISS,"Contas que registram o Imposto sobre Serviço de qualquer Natureza (ISS) relativo às receitas de serviços, conforme legislação específica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050101030600,3.05.01.01.03.06.00,(-) Demais Impostos e Contribuições Incidentes sobre Vendas e Serviços,"Contas que registrem os demais impostos e contribuições incidentes sobre as receitas das vendas de que trata a conta Receita da Atividade Rural, que guardem proporcionalidade com o preço e sejam considerados redutores das receitas de vendas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050103010000,3.05.01.03.01.00.00,Custo dos Produtos Vendidos da Atividade Rural,,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010100,3.05.01.05.01.01.00,Variações Cambiais Ativas,"Contas que registram os ganhos apurados em razão de variações ativas decorrentes da atualização dos direitos de crédito e obrigações, calculados com base nas variações nas taxas de câmbio.
Atenção:
1) as variações cambiais ativas decorrentes dos direitos de crédito e de obrigações, em função da taxa de câmbio, são consideradas como receita financeira, inclusive para fins de cálculo do lucro da exploração (Lei no 9.718, art. 9o c/c art. 17);
2) nas atividades de compra e venda, loteamento, incorporação e construção de imóveis, as variações cambiais ativas são reconhecidas como receita segundo as normas constantes da IN SRF no 84/79, de 20 de dezembro de 1979, da IN SRF no 23/83, de 25 de março de 1983, e da IN SRF no 67/88, de 21 de abril de 1988 (IN SRF no 25/99, de 25 de fevereiro de 1999).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010200,3.05.01.05.01.02.00,"Ganhos Auferidos no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório dos ganhos auferidos, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País; 
b) os ganhos auferidos nas alienações, fora de bolsa, de ouro, ativo financeiro, e de participações societárias, exceto as alienações de participações societárias permanentes em sociedades coligadas e controladas e de participações societárias que permanecerem no ativo da pessoa jurídica até o término do ano-calendário seguinte ao de suas aquisições; e 
c) os rendimentos auferidos em operações de swap e no resgate de quota de fundo de investimento cujas carteiras sejam constituídas, no mínimo, por 67% (sessenta e sete por cento) de ações no mercado à vista de bolsa de valores ou entidade assemelhada (Lei no 9.532, de 1997, art. 28, alterado pela MP no 1.636, de 1998, art. 2o, e reedições).
Considera-se ganho o resultado positivo auferido nas operações citadas acima, realizadas em cada mês, admitida a dedução dos custos e despesas incorridos, necessários à realização das operações.
Atenção:
1) os ganhos auferidos em operações day-trade devem ser informados em conta específica;
2) o valor correspondente às perdas incorridas no mercado de renda variável, exceto day-trade, deve ser informado em conta específica. 
3) são consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros as entidades cujo objeto social seja análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010300,3.05.01.05.01.03.00,Ganhos em Operações Day-Trade,"Contas que registram os ganhos diários auferidos, em cada mês do período de apuração, em operações day-trade. Considera-se ganho o resultado positivo auferido nas operações citadas acima, realizadas em cada mês, admitida a dedução dos custos e despesas incorridos, necessários à realização das operações. Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado à vista, no mesmo dia. Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia.
Atenção: o valor correspondente às perdas incorridas nas operações day-trade deve ser informado em conta específica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010400,3.05.01.05.01.04.00,Receitas de Juros sobre o Capital Próprio,"Contas que registram os juros recebidos, a título de remuneração do capital próprio, em conformidade com o art. 9o da Lei no 9.249, de 1995. O valor  informado deve corresponder ao total dos juros recebidos antes do desconto do imposto de renda na fonte. O valor do imposto de renda retido na fonte, para as pessoas jurídicas tributadas pelo lucro real, é considerado antecipação do imposto devido no encerramento do período de apuração ou, ainda, pode ser compensado com aquele que for retido, pela beneficiária, por ocasião do pagamento ou crédito de juros a título de remuneração do capital próprio, ao seu titular ou aos seus sócios.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010500,3.05.01.05.01.05.00,Outras Receitas Financeiras,"Contas que registram receitas auferidas no período de apuração relativas a juros, descontos, lucro na operação de reporte, prêmio de resgate de títulos ou debêntures e rendimento nominal auferido em aplicações financeiras de renda fixa, não incluídas em contas precedentes deste grupo. As receitas dessa natureza, derivadas de operações com títulos vencíveis após o encerramento do período de apuração, serão rateadas segundo o regime de competência.
Atenção:
1) as variações monetárias ativas decorrentes da atualização dos direitos de crédito e das obrigações, em função de índices ou coeficientes aplicáveis por disposição legal ou contratual, devem ser informadas como receita financeira;
2) As variações cambiais ativas devem ser informadas na conta Variações Cambiais Ativas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010600,3.05.01.05.01.06.00,Ganhos na Alienação de Participações Não Integrantes do Ativo Permanente,"Contas que registram os ganhos auferidos na alienação de ações, títulos ou quotas de capital não integrantes do ativo permanente, desde que não incluídos na conta Ganhos Auferidos no Mercado de Renda Variável, exceto Day-Trade.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010700,3.05.01.05.01.07.00,Resultados Positivos em Participações Societárias,"Contas que registram:
a) os lucros e dividendos derivados de investimentos avaliados pelo custo de aquisição;
b) os ganhos por ajustes no valor de investimentos relevantes avaliados pelo método da equivalência patrimonial, decorrentes de lucros apurados nas controladas e coligadas;
Atenção: considera-se controlada a filial, a agência, a sucursal, a dependência ou o escritório de representação no exterior, sempre que os respectivos ativos e account.data_account_type_current_liabilitiess não estejam incluídos na contabilidade da investidora, por força de normatização específica.
c) as bonificações recebidas; 
Atenção:
1) as bonificações recebidas, decorrentes da incorporação de lucros ou reservas não tributados na forma do art. 35 da Lei no 7.713, de 1988, ou apurados nos anos-calendário de 1994 ou 1995, são consideradas a custo zero, não afetando o valor do investimento nem o resultado do período de apuração (art. 3o da Lei no 8.849, de 1994, e art. 3o da Lei no 9.064, de 1995).;
2) no caso de investimento avaliado pelo custo de aquisição, as bonificações recebidas, decorrentes da incorporação de lucros ou reservas tributados na forma do art. 35 da Lei no 7.713, de 1988, e de lucros ou reservas apurados no ano-calendário de 1993 ou a partir do ano-calendário de 1996, são registradas tomando-se como custo o valor da parcela dos lucros ou reservas capitalizados.
e) os lucros e dividendos de participações societárias avaliadas pelo custo de aquisição;
Atenção:os lucros ou dividendos recebidos em decorrência de participações societárias avaliadas pelo custo de aquisição adquiridas até 6 (seis) meses antes da data do recebimento devem ser registrados como diminuição do valor do custo, não sendo incluídos nesta conta.
f) os resultados positivos decorrentes de participações societárias no exterior avaliadas pelo patrimônio líquido, os dividendos de participações avaliadas pelo custo de aquisição e os resultados de equivalência patrimonial relativos a filiais, sucursais ou agências da pessoa jurídica localizadas no exterior, em decorrência de operações realizadas naquelas filiais, sucursais ou agências.Os lucros auferidos no exterior serão adicionados ao lucro líquido, para efeito de determinação do lucro real, no período de apuração correspondente ao balanço levantado em 31 de dezembro do ano-calendário em que tiverem sido disponibilizados, observando-se o disposto nos arts. 394 e 395 do Decreto no 3.000, de 1999, e no art. 74 da Medida Provisória no 2.158-35, de 24 de agosto de 2001.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010710,3.05.01.05.01.07.10,Amortização de Deságio nas Aquisições de Investimentos Avaliados pelo Patrimônio Líquido,"Contas que registram as amortizações de deságios nas aquisições de investimentos avaliados pelo patrimônio líquido. O valor amortizado que for excluído do lucro líquido para determinação do lucro real deve ser controlado na Parte B do Livro de Apuração do Lucro Real até a alienação ou baixa da participação societária, quando, então, deve ser adicionado ao lucro líquido para determinação do lucro real no período de apuração em que for computado o ganho ou perda de capital havido.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010800,3.05.01.05.01.08.00,Resultados Positivos em SCP,"Conta utilizada pelas pessoas jurídicas que forem sócias ostensivas de sociedades em conta de participação, para a registro:
a) de lucros derivados de participação em SCP, avaliadas pelo custo de aquisição;
b) dos ganhos por ajustes no valor de participação em SCP, avaliadas pelo método da equivalência patrimonial.
Atenção:os lucros recebidos de investimento em SCP, avaliado pelo custo de aquisição, ou a contrapartida do ajuste do investimento ao valor do patrimônio líquido da SCP, no caso de investimento avaliado por esse método, podem ser excluídos na determinação do lucro real dos sócios, pessoas jurídicas, das referidas sociedades (Decreto no 3.000, de 1999, art. 149).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105010900,3.05.01.05.01.09.00,Rendimentos e Ganhos de Capital Auferidos no Exterior,"Contas que registram os rendimentos e ganhos de capital auferidos no exterior diretamente pela pessoa jurídica domiciliada no Brasil, pelos seus valores antes de descontado o tributo pago no país de origem. Esses valores podem, no caso de apuração trimestral do imposto, ser excluídos na apuração do lucro real do 1o ao 3o trimestres, devendo ser adicionados ao lucro líquido na apuração do lucro real referente ao 4o trimestre. 
Atenção:Os ganhos de capital referentes a alienações de bens e direitos do ativo permanente situados no exterior devem ser informados na conta Outras Receitas Não Operacionais.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105011000,3.05.01.05.01.10.00,Reversão dos Saldos das Provisões Operacionais,"Contas que registram a reversão de  saldos não utilizados das provisões constituídas no balanço do período de apuração imediatamente anterior para fins de apuração do lucro real (Lei no 9.430, de 1996, art. 14).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105011100,3.05.01.05.01.11.00,Outras Receitas Operacionais,"Contas que registram todas as demais receitas que, por definição legal, sejam consideradas operacionais, tais como:
a) aluguéis de bens por empresa que não tenha por objeto a locação de móveis e imóveis;
b) recuperações de despesas operacionais de períodos de apuração anteriores, tais como: prêmios de seguros, importâncias levantadas das contas vinculadas do FGTS, ressarcimento de desfalques, roubos e furtos, etc. As recuperações de custos e despesas no decurso do próprio período de apuração devem ser creditadas diretamente às contas de resultado em que foram debitadas;
c) os créditos presumidos do IPI para ressarcimento do valor da Contribuição ao PIS/Pasep e Cofins;
d) multas ou vantagens a título de indenização em virtude de rescisão contratual (Lei no 9.430, de 1996, art. 70, § 3o, II);e) o crédito presumido da contribuição para o PIS/Pasep e da Cofins concedido na forma do art. 3o da Lei no 10.147, de 2000.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105011200,3.05.01.05.01.12.00,Prêmios Recebidos na Emissão de Debêntures,"Contas que registram, a partir de 01.01.2008, os prêmios recebidos na emissão de debêntures.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105011300,3.05.01.05.01.13.00,Doações e Subvenções para Investimentos,"Contas que registram, a partir de 01.01.2008, as doações e subvenções para investimento.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105011400,3.05.01.05.01.14.00,Contrapartida dos Ajustes ao Valor Presente,"Contrapartida do ajuste ao valor presente dos elementos do ativo e do account.data_account_type_current_liabilities (art. 183, inciso VIII, e art. 184, inciso III da Lei 6.404/76)",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050105011500,3.05.01.05.01.15.00,Contrapartida de outros Ajustes às Normas Internacionais de Contabilidade,Contrapartida de outros ajustes decorrentes da adequação às Normas Internacionais de Contabilidade,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010100,3.05.01.07.01.01.00,Remuneração a Dirigentes e a Conselho de Administração,"Contas que registram a despesa incorrida relativa à remuneração mensal e fixa atribuída ao titular de firma individual, aos sócios, diretores e administradores de sociedades, ou aos representantes legais de sociedades estrangeiras, as despesas incorridas com os salários indiretos concedidos pela empresa a administradores, diretores, gerentes e seus assessores (PN Cosit no 11, de 1992), e o valor referente às remunerações atribuídas aos membros do conselho fiscal ou consultivo.
Atenção: os valores das gratificações aos dirigentes que estejam ligados à área de produção rural devem ser informados na conta Remuneração a Dirigentes da Produção.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010201,3.05.01.07.01.02.01,"Ordenados, Salários Gratificações e Outras Remunerações a Empregados","Contas que registram as despesas com ordenados, salários, gratificações e outras despesas com empregados, tais como: comissões, moradia, seguro de vida e outras de caráter remuneratório.
Atenção:
1) As despesas correspondentes a salários, ordenados, gratificações e outras remunerações referentes à área de saúde, tais como assistência médica, odontológica e farmacêutica, devem ser indicadas na conta específica.
2) Não deve ser informado nesta conta o valor referente às participações dos empregados no lucro da pessoa jurídica. Esse valor deve ser informado na conta Participações de Empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010203,3.05.01.07.01.02.03,Planos de Poupança e Investimentos de Empregados,Contas que registram o valor total dos gastos efetuados com Planos de Poupança e Investimentos (PAIT).,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010205,3.05.01.07.01.02.05,Fundo de Aposentadoria Programada Individual de Empregados,Contas que registram o valor total dos gastos efetuados com Fundos de Aposentadoria Programada Individual (FAPI).,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010207,3.05.01.07.01.02.07,Plano de Previdência Privada de Empregados,Contas que registram o valor total dos gastos efetuados com Planos de Previdência Privada.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010209,3.05.01.07.01.02.09,Outros Gastos com Pessoal,"Contas que registram os gastos com empregados não enquadrados nas contas precedentes
Atenção: 
1) As despesas correspondentes a salários, ordenados, gratificações e outras remunerações referentes à área de saúde, tais como assistência médica, odontológica e farmacêutica, devem ser indicadas na conta  Assistência Médica, Odontológica e Farmacêutica a Empregados;
2) não deve ser informado nesta conta o valor referente às participações dos empregados no lucro da pessoa jurídica. Esse valor deve ser informado na conta Participações de Empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010300,3.05.01.07.01.03.00,Prestação de Serviços por Pessoa Física sem Vínculo Empregatício,"Contas que registram, salvo se houver conta mais específica, as despesas correspondentes aos serviços prestados por pessoa física que não tenha vínculo empregatício com a pessoa jurídica, tais como: comissões, corretagens, gratificações, honorários, direitos autorais e outras remunerações, inclusive as relativas a empreitadas de obras exclusivamente de trabalho e as decorrentes de fretes e carretos em geral.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010400,3.05.01.07.01.04.00,Prestação de Serviço Pessoa Jurídica,"Contas que registram, salvo se houver conta mais específica, o valor das despesas correspondentes aos serviços prestados por outra pessoa jurídica à pessoa jurídica declarante.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010401,3.05.01.07.01.04.01,Serviços Prestados por Cooperativa de Trabalho,Contas que registram os serviços prestados por cooperativa de trabalho,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010402,3.05.01.07.01.04.02,Locação de Mão-de-obra,"Contas que registram o valor total dos gastos efetuados no período com a contratação de serviços executados mediante cessão de mão-de-obra ou empreitada, inclusive em regime temporário, sujeitos à retenção de contribuição previdenciária, nos termos do art. 219 do Regulamento da Previdência Social - RPS, aprovado pelo Decreto nº 3.048, de 1999",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010500,3.05.01.07.01.05.00,Encargos Sociais - Previdência Social,"Contas que registram as contribuições para a Previdência Social, não computadas nos custos (inclusive dos dirigentes - PN CST no 35, de 31 de agosto de 1981).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010600,3.05.01.07.01.06.00,Encargos Sociais – FGTS,"Contas que registram as contribuições para a o FGTS, não computadas nos custos (inclusive dos dirigentes - PN CST no 35, de 31 de agosto de 1981).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010700,3.05.01.07.01.07.00,Encargos Sociais – Outros,"Contas que registram os demais encargos sociais, não computados nos custos ou nas contas Encargos Sociais - Previdência Social ou Encargos Sociais - FGTS",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010800,3.05.01.07.01.08.00,Doações e Patrocínios de Caráter Cultural e Artístico (Lei no 8.313/1991),"Contas que registram as doações e patrocínios efetuados no período de apuração em favor de projetos culturais previamente aprovados pelo Ministério da Cultura ou pela Agência Nacional do Cinema (Ancine), observada a legislação de concessão dos projetos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107010900,3.05.01.07.01.09.00,"Doações a Instituições de Ensino e Pesquisa (Lei no 9.249/1995, art.13, § 2o)","Contas que registram as doações a instituições de ensino e pesquisa cuja criação tenha sido autorizada por lei federal e que preencham os requisitos dos incisos I e II do art. 213 da Constituição Federal, de 1988, que são:
a) comprovação de finalidade não-lucrativa e aplicação dos excedentes financeiros em educação;
b) assegurar a destinação do seu patrimônio a outra escola comunitária, filantrópica ou confessional, ou ao Poder Público, no caso de encerramento de suas atividades.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011000,3.05.01.07.01.10.00,Doações a Entidades Civis,"Contas que registram as doações efetuadas a:
a) entidades civis, legalmente constituídas no Brasil, sem fins lucrativos, que prestem serviços gratuitos em benefício de empregados da pessoa jurídica doadora, e respectivos dependentes, ou em benefício da comunidade na qual atuem; e
b) Organizações da Sociedade Civil de Interesse Público (OSCIP), qualificadas segundo as normas estabelecidas na Lei no 9.790, de 23 de março de 1999.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011100,3.05.01.07.01.11.00,Outras Contribuições e Doações,"Contas que registram as doações feitas, entre outras, aos Fundos controlados pelos Conselhos Municipais, Estaduais e Nacional dos Direitos da Criança e do Adolescente.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011200,3.05.01.07.01.12.00,Alimentação do Trabalhador,"Contas que registram as despesas com alimentação do pessoal não ligado à produção, realizadas durante o período de apuração, ainda que a pessoa jurídica não tenha Programa de Alimentação do Trabalhador aprovado pelo Ministério do Trabalho.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011300,3.05.01.07.01.13.00,PIS/Pasep,Contas que registram as Contribuições para o PIS/Pasep incidente sobre as demais receitas operacionais.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011400,3.05.01.07.01.14.00,Cofins,Contas que registram a parcela da Cofins incidente sobre as demais receitas operacionais.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011500,3.05.01.07.01.15.00,CPMF,Contas que registram a Contribuição Provisória sobre Movimentação ou Transmissão de Valores e de Créditos de Natureza Financeira.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011600,3.05.01.07.01.16.00,"Demais Impostos, Taxas e Contribuições, exceto IR e CSLL","Contas que registram os demais Impostos, Taxas e Contribuições, exceto: 
a) incorporadas ao custo de bens do ativo permanente;
b) correspondentes aos impostos não recuperáveis, incorporados ao custo das matérias-primas, materiais secundários, materiais de embalagem e mercadorias destinadas à revenda;
c) correspondentes aos impostos recuperáveis;
d) correspondentes aos impostos e contribuições redutores da receita bruta ;
e) correspondentes às Contribuições para o PIS/Pasep e à Cofins incidentes sobre as demais receitas operacionais, e à CPMF, indicados em contas específicas;
f) correspondentes à contribuição social sobre o lucro líquido e ao imposto de renda devidos, que são informados em contas específicas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011700,3.05.01.07.01.17.00,Arrendamento Mercantil,"Contas que registram as despesas, não computadas nos custos, pagas ou creditadas a título de contraprestação de arrendamento mercantil, decorrentes de contrato celebrado com observância da Lei no 6.099, de 12 de setembro de 1974, com as alterações da Lei no 7.132, de 26 de outubro de 1983, e da Portaria MF no 140, de 1984",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011800,3.05.01.07.01.18.00,Aluguéis,Contas que registram as despesas com aluguéis não decorrentes de arrendamento mercantil.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107011900,3.05.01.07.01.19.00,Despesas com Veículos e de Conservação de Bens e Instalações,"Contas que registram as despesas relativas aos bens que não estejam ligados diretamente à produção, as realizadas com reparos que não impliquem aumento superior a um ano da vida útil do bem, prevista no ato de sua aquisição, e as relativas a combustíveis e lubrificantes para veículos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012001,3.05.01.07.01.20.01,"Propaganda, Publicidade e Patrocínio (Associações Desportivas que Mantenham Equipe de Futebol Profissional)","Contas que registram as despesas relativas a propaganda publicidade e patrocínio com associações desportivas que mantenham equipe de futebol profissional e possuam registro na Federação de Futebol do respectivo Estado, a título de propaganda, publicidade e patrocínio.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012002,3.05.01.07.01.20.02,"Propaganda, Publicidade e Patrocínio","Contas que registram de propaganda, publicidade, exceto as classificadas na conta precedente",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012100,3.05.01.07.01.21.00,Multas,Contas que registram as despesas com multas.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012200,3.05.01.07.01.22.00,Encargos de Depreciação e Amortização,"Contas que registram apenas os encargos a esses títulos, com bens não aplicados diretamente na produção. Inclui a amortização dos ajustes de variação cambial contabilizada no ativo diferido, relativa à atividade geral da pessoa jurídica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012300,3.05.01.07.01.23.00,Perdas em Operações de Crédito,Contas que registram as perdas no recebimento de créditos decorrentes das atividades da pessoa jurídica.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012400,3.05.01.07.01.24.00,Provisões para Férias e 13o Salário de Empregados,"Contas que registram as despesas com a constituição de provisões para:
a) pagamento de remuneração correspondente a férias e adicional de férias de empregados, inclusive encargos sociais (Decreto no 3.000, de 1999, art. 337, e PN CST no 7, de 1980); 
b) o 13o salário, no caso de apuração trimestral do imposto, inclusive encargos sociais (Decreto no 3.000, de 1999, art. 338).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012500,3.05.01.07.01.25.00,Provisão para Perda de Estoque,Contas que registram as despesas com a constituição de provisão para perda de estoque,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012600,3.05.01.07.01.26.00,Demais Provisões,Contas que registram as despesas com provisões não relacionadas em contas específicas,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012700,3.05.01.07.01.27.00,Gratificações a Administradores,Contas que registram as gratificações a administradores.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012800,3.05.01.07.01.28.00,Royalties e Assistência Técnica – PAÍS,"Contas que registram as despesas correspondentes às importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no Brasil, a título de royalties e assistência técnica, científica ou assemelhada, que não estejam relacionados com a produção.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107012900,3.05.01.07.01.29.00,Royalties e Assistência Técnica – EXTERIOR,"Contas que registram as despesas correspondentes às importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no exterior, a título de royalties e assistência técnica, científica ou assemelhada, que não estejam relacionados com a produção.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107013000,3.05.01.07.01.30.00,"Assistência Médica, Odontológica e Farmacêutica a Empregados","Indicar o valor das despesas com assistência médica, odontológica e farmacêutica. 
Atenção: o valor referente à contratação de serviços de profissionais liberais sem vínculo empregatício ou de sociedades civis deve ser informado nas contas Prestação de Serviços por Pessoa Física sem Vínculo Empregatício ou Prestação de Serviço Pessoa Jurídica, conforme o caso.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107013100,3.05.01.07.01.31.00,Pesquisas Científicas e Tecnológicas,"Contas que registram as despesas efetuadas a esse título, inclusive a contrapartida das amortizações daquelas registradas no ativo diferido",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107013200,3.05.01.07.01.32.00,Bens de Natureza Permanente Deduzidos como Despesa,"Contas que registram as despesas com aquisição de bens do ativo imobilizado cujo prazo de vida útil não ultrapasse um ano, ou, caso exceda esse prazo, tenha valor unitário igual ou inferior ao fixado no art. 301 do Decreto no 3.000, de 1999.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107013301,3.05.01.07.01.33.01,"Despesas com viagens, diárias e ajusta de custo","Contas que registram as despesas operacionais com viagens, diárias e ajuda de custo",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050107013390,3.05.01.07.01.33.90,Outras Despesas Operacionais,"Contas que registram as demais despesas operacionais, cujos títulos não se adaptem à nomenclatura específica desta ficha, tais como:
a) contribuição sindical;
b) prêmios de seguro;
c) fretes e carretos que não componham os custos;
d) transporte de empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010100,3.05.01.09.01.01.00,(-) Variações Cambiais Passivas,"Contas que registram as perdas monetárias passivas resultantes da atualização dos direitos de créditos e das obrigações, calculadas com base nas variações nas taxas de câmbio (Lei no 9.069, de 1995, art. 52, e Lei no 9.249, de 1995, art. 8o).Inclusive a variação cambial passiva correspondente: 
a) à atualização das obrigações e dos créditos em moeda estrangeira, registrada em qualquer data e apurada no encerramento do período de apuração em função da taxa de câmbio vigente;
b) às operações com moeda estrangeira e conversão de obrigações para moeda nacional, ou novação dessas obrigações, ou sua extinção, total ou parcial, em virtude de capitalização,dação em pagamento, compensação, ou qualquer outro modo, desde que observadas as condições fixadas pelo Banco Central do Brasil.
Atenção: a amortização dos ajustes de variação cambial contabilizada no ativo 
diferido deve ser informada na conta Encargos de Depreciação e Amortização (Lei no 9.816, de 1999, art. 2o, e Lei no 10.305, de 2001).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010200,3.05.01.09.01.02.00,"(-) Perdas Incorridas no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório das perdas incorridas, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País;
b) as perdas incorridas nas alienações, fora de bolsa, de ouro, ativo financeiro, e de participações societárias, exceto as alienações de participações societárias permanentes em sociedades coligadas e controladas e de participações societárias que permanecerem no ativo da pessoa jurídica até o término do ano-calendário seguinte ao de suas aquisições; e
c) as perdas em operações de swap e no resgate de quota de fundo de investimento que mantenha, no mínimo, 67% (sessenta e sete por cento) de ações negociadas no mercado à vista de bolsa de valores ou entidade assemelhada (Lei no 9.532, de 1997, art. 28, alterado pela MP no 1.636, de 1998, art. 2o, e reedições).
São consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros as entidades cujo objeto social seja análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM).
Atenção: as perdas apuradas em operações day-trade devem ser informadas em conta própria.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010300,3.05.01.09.01.03.00,(-) Perdas em Operações Day-Trade,"Contas que registram o somatório das perdas diárias apuradas, em cada mês do período de apuração, em operações day-trade.Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado à vista, no mesmo dia.Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010400,3.05.01.09.01.04.00,(-) Juros sobre o Capital Próprio,"Contas que registram as despesas com juros pagos ou creditados individualizadamente a titular, sócios ou acionistas, a título de remuneração do capital próprio, calculados sobre as contas do patrimônio líquido e limitados à variação, pro rata dia, da Taxa de Juros de Longo Prazo (TJLP), observando-se o regime de competência (Lei no 9.249, de 1995, art. 9o).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010500,3.05.01.09.01.05.00,(-) Outras Despesas Financeiras,"Contas que registram as despesas relativas a juros, não incluídas nas em outras contas, a descontos de títulos de crédito e ao deságio na colocação de debêntures ou outros títulos. Tais despesas serão obrigatoriamente rateadas, segundo o regime de competência.
Atenção:
1) as variações monetárias passivas decorrentes da atualização das obrigações, em função de índices ou coeficientes aplicáveis por disposição legal ou contratual, devem ser informadas como despesa financeira.
2) As variações cambiais passivas não devem ser informadas nesta conta, e sim na conta Variações Cambiais Passivas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010600,3.05.01.09.01.06.00,(-) Prejuízos na Alienação de Participações Não Integrantes do Ativo Permanente,"Contas que registram os prejuízos havidos em virtude de alienação de ações, títulos ou quotas de capital não integrantes do ativo permanente, desde que não incluídos nas contas Perdas Incorridas no Mercado de Renda Variável, exceto Day-Trade ou Perdas em Operações Day-Trade.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010700,3.05.01.09.01.07.00,(-) Resultados Negativos em Participações Societárias,"Contas que registram as perdas por ajustes no valor de investimentos relevantes avaliados pelo método da equivalência patrimonial, decorrentes de prejuízos apurados nas controladas e coligadas. 
Atenção: considera-se controlada a filial, a agência, a sucursal, a dependência ou o escritório de representação no exterior, sempre que os respectivos ativos e account.data_account_type_current_liabilitiess não estejam incluídos na contabilidade da investidora, por força de normatização específica.
Devem, também, ser indicados nesta conta os resultados negativos derivados de participações societárias no exterior, avaliadas pelo patrimônio líquido. Incluem-se, nestas informações, as perdas apuradas em filiais, sucursais e agências da pessoa jurídica localizadas no exterior.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010710,3.05.01.09.01.07.10,(-) Amortização de Ágio nas Aquisições de Investimentos Avaliados pelo Patrimônio Líquido,"Contas que registram o valor da amortização registrada no período, referente ao ágio nas aquisições de investimentos avaliados pelo método da equivalência patrimonial.
Atenção: O valor amortizado deve ser adicionado ao lucro líquido, para determinação do lucro real, e controlado na Parte B do Livro de Apuração do Lucro Real até a alienação ou baixa da participação societária, quando, então, pode ser excluído do lucro líquido, para determinação do lucro real.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010800,3.05.01.09.01.08.00,(-) Resultados Negativos em SCP,"Conta utilizada pelos sócios ostensivos, pessoas jurídicas, de sociedades em conta de participação, para indicar as perdas por ajustes no valor de participação em SCP, avaliada pelo método da equivalência patrimonial.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109010900,3.05.01.09.01.09.00,(-) Perdas em Operações Realizadas no Exterior,"Contas que registram as perdas em operações realizadas no exterior diretamente pela pessoa jurídica domiciliada no Brasil, com exceção das perdas de capital decorrentes da alienação de bens e direitos do ativo permanente situados no exterior, que devem ser indicadas na conta Outras Despesas Não Operacionais.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109011000,3.05.01.09.01.10.00,(-) Contrapartida dos Ajustes ao Valor Presente,"Contrapartida do ajuste ao valor presente dos elementos do ativo e do account.data_account_type_current_liabilities (art. 183, inciso VIII, e art. 184, inciso III da Lei 6.404/76)",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109011100,3.05.01.09.01.11.00,(-) Contrapartida de outros Ajustes às Normas Internacionais de Contabilidade,Contrapartida de outros ajustes decorrentes da adequação às Normas Internacionais de Contabilidade,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050109011200,3.05.01.09.01.12.00,(-) Contrapartida dos ajustes de valor do imobilizado e intangível,"Contrapartida dos ajustes decorrentes da análise de recuperação dos valores registrados no imobilizado e no  intangível (art. 183, § 3º, da Lei 6.404/76)",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050301010100,3.05.03.01.01.01.00,(-) Participações de Empregados,"Contas que registram as participações atribuídas a empregados segundo disposição legal, estatutária, contratual ou por deliberação da assembléia de acionistas ou sócios.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050301010200,3.05.03.01.01.02.00,(-) Contribuições para Assistência ou Previdência de Empregados,"Contas que registram as contribuições para instituições ou fundos de assistência ou previdência de empregados, baseadas nos lucros. Não indicar, nesta conta, aquelas contribuições já deduzidas como custo ou despesa operacional.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050301010300,3.05.03.01.01.03.00,(-) Outras Participações de Empregados,Contas que registram outras participações de empregados,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050301030100,3.05.03.01.03.01.00,(-) Participações de Administradores e Partes Beneficiárias,"Contas que registram quaisquer participações nos lucros atribuídas a administradores, sócio, titular de empresa individual e a portadores de partes beneficiárias, durante o período de apuração.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050301030200,3.05.03.01.03.02.00,(-) Participações de Debêntures,Contas que representam as participações nos lucros da companhia atribuídas a debêntures de sua emissão,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3050301030500,3.05.03.01.03.05.00,(-) Outras ,Contas que registram outras participações,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3060101010100,3.06.01.01.01.01.00,(-) Contribuição Social sobre o Lucro Líquido,Contas que registram as provisões para a CSLL calculadas sobre a base de cálculo correspondente ao período de apuração e sobre os lucros diferidos da atividade rural.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_3060101010200,3.06.01.01.01.02.00,(-) Provisão para Imposto de Renda - Pessoa Jurídica,Contas que registram as provisões para o IRPJ calculadas sobre a base de cálculo correspondente ao período de apuração e sobre os lucros diferidos da atividade rural.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101010100,4.01.01.01.01.01.00,Da atividade de Educação,Contas que registram a receita de venda dos produtos da atividade de educação.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101010200,4.01.01.01.01.02.00,Da atividade de Saúde,Contas que registram a receita de venda dos produtos da atividade de saúde.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101010300,4.01.01.01.01.03.00,Da atividade de Assistência Social,Contas que registram a receita de venda dos produtos da atividade de assistência social.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101010400,4.01.01.01.01.04.00,Outras,Contas que registram as demais receitas de vendas de produtos.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101020100,4.01.01.01.02.01.00,Serviços Educacionais,Contas que registram as receitas de prestação de serviços na atividade educacional.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101020200,4.01.01.01.02.02.00,Doações/Subvenções Vinculadas ,"Contas que registram as receitas recebidas como doações/subvenções vinculadas (Dec. 2.536/1998, art. 3, inciso V),  com destinação à prestação de serviços, preferencialmente segregadas por níveis federal, estadual e municipal.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101020300,4.01.01.01.02.03.00,Doações,"Contas que registram as receitas recebidas como doações particulares não vinculadas,  com destinação à prestação de serviços.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101020400,4.01.01.01.02.04.00,Contribuições,Contas que registram as receitas recebidas como contribuições com destinação à prestação de serviços.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101020500,4.01.01.01.02.05.00,Outras,Contas que registram as demais receitas de prestação de serviços.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101030100,4.01.01.01.03.01.00,Pacientes Particulares,Contas que registram as receitas de serviços de saúde prestados a pacientes particulares.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101030200,4.01.01.01.03.02.00,Convênios – SUS,Contas que registram as receitas de serviços de saúde prestados a pacientes conveniados do SUS.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101030300,4.01.01.01.03.03.00,Convênios – Outros,Contas que registram as receitas de serviços de saúde prestados a outros pacientes conveniados.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101030400,4.01.01.01.03.04.00,Doações/Subvenções Vinculadas,"Contas que registram as receitas recebidas como doações/subvenções vinculadas (Dec. 2.536/1998, art. 3, inciso V),  com destinação à área de saúde, preferencialmente segregadas por níveis federal, estadual e municipal.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101030500,4.01.01.01.03.05.00,Doações,"Contas que registram as receitas recebidas como doações particulares não vinculadas,  com destinação à área da saúde.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101030600,4.01.01.01.03.06.00,Contribuições,Contas que registram as receitas recebidas como contribuições com destinação à área de saúde.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101030700,4.01.01.01.03.07.00,Outras,Contas que registram as demais receitas de serviços de saúde.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101040100,4.01.01.01.04.01.00,Pacientes Particulares,Contas que registram as receitas de serviços na área de assistência social a pacientes particulares.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101040200,4.01.01.01.04.02.00,Convênios - Outros,Contas que registram as receitas de serviços na área de assistência social a pacientes particulares através de convênios/contratos/termos de parcerias.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101040300,4.01.01.01.04.03.00,Doações/Subvenções Vinculadas,"Contas que registram as receitas recebidas como doações/subvenções vinculadas (Dec. 2.536/1998, art. 3, inciso V),  com destinação à área de assistência social, preferencialmente segregadas por níveis federal, estadual e municipal.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101040400,4.01.01.01.04.04.00,Doações,"Contas que registram as receitas recebidas como doações particulares não vinculadas,  com destinação à área de assistência social.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101040500,4.01.01.01.04.05.00,Contribuições,Contas que registram as receitas recebidas como contribuições com destinação à área de assistência social.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101040600,4.01.01.01.04.06.00,Outras,Contas que registram as demais receitas de serviços na área de assistência social.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101050100,4.01.01.01.05.01.00,Contribuições Sindicais,Contas que registram receitas com a natureza de contribuições sindicais.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101050200,4.01.01.01.05.02.00,Contribuições Confederativas/Associativas,Contas que registram receitas com a natureza de contribuições confederativas e/ou associativas.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101050300,4.01.01.01.05.03.00,Mensalidades,Contas que registram receitas com a natureza de mensalidades revertidas por seus associados.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101050400,4.01.01.01.05.04.00,Doações/Subvenções,"Contas que registram receitas com a natureza de doações e/ou subvenções recebidas de entidades públicas e/ou privadas, e de pessoas físicas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101050500,4.01.01.01.05.05.00,Outras Contribuições,Demais contas que registram contribuições não especificadas anteriormente.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101050600,4.01.01.01.05.06.00,Outras, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101090100,4.01.01.01.09.01.00,(-) Vendas Canceladas,Contas que registram vendas das prestações de serviços canceladas.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101090200,4.01.01.01.09.02.00,(-) Devoluções e Descontos Incondicionais,Contas que registram as devoluções e descontos incondicionais nas atividades da entidade.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010101090300,4.01.01.01.09.03.00,Outras,Contas que registram as demais deduções da receita bruta.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301010100,4.01.03.01.01.01.00,Custos dos Produtos para Educação - Vendidos,Contas que registram o custo do produto vendido na área de educação.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301010200,4.01.03.01.01.02.00,Custos dos Produtos para Educação - Gratuidades,Contas que registram o custo do produto dado em gratuidade na área de educação.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301010300,4.01.03.01.01.03.00,Outros Custos, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301020100,4.01.03.01.02.01.00,Custos dos Produtos para Saúde – Vendidos,Contas que registram o custo do produto vendido na área de saúde.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301020200,4.01.03.01.02.02.00,Custos dos Produtos para Saúde - Gratuidades,Contas que registram o custo do produto dado em gratuidade na área de saúde.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301020300,4.01.03.01.02.03.00,Outros Custos, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301030100,4.01.03.01.03.01.00,Custos dos Produtos para Assistência Social - Vendidos,Contas que registram o custo do produto vendido na área de assistência social.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301030200,4.01.03.01.03.02.00,Custos dos Produtos para Assistência Social - Gratuidades,Contas que registram o custo do produto dado em gratuidade na área de assistência social.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301030300,4.01.03.01.03.03.00,Outras, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301040100,4.01.03.01.04.01.00,Custos dos Produtos Vendidos em Geral,Contas que registram o custo do produto vendido nas atividades não abrangidas anteriormente.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010301040200,4.01.03.01.04.02.00,Outros Custos, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302010100,4.01.03.02.01.01.00,Custo dos Serviços Prestados a Alunos Não Bolsistas,Contas que registram o custo da prestação do serviço para os alunos não bolsistas.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302010200,4.01.03.02.01.02.00,Custo dos Serviços Prestados a Convênios/Contratos/Parcerias (Exceto PROUNI),"Contas que registram o custo da prestação do serviço para os alunos vinculados aos convênios/contratos/parcerias, exceto àqueles que estão no PROUNI.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302010300,4.01.03.02.01.03.00,Custo dos Serviços Prestados a Doações/Subvenções Vinculadas,"Contas que registram o custo da prestação do serviço para os alunos vinculados à doações/subvenções vinculadas (Dec. 2.536/1998, art. 3, inciso V),  com destinação à área de educação, preferencialmente segregadas por níveis federal, estadual e municipal.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302010400,4.01.03.02.01.04.00,Custo dos Serviços Prestados a Doações,"Contas que registram o custo da prestação do serviço para os alunos vinculados às demais doações,  com destinação à área de educação, exceto àquelas doações vinculadas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302010500,4.01.03.02.01.05.00,Custo dos Serviços Prestados ao PROUNI,Contas que registram o custo da prestação do serviço para os alunos vinculados ao PROUNI.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302010600,4.01.03.02.01.06.00,Custo dos Serviços Prestados a Gratuidade ,"Contas que registram o custo da prestação do serviço para os alunos com gratuidades de bolsas parciais e/ou integrais, exceto às vinculadas ao PROUNI, sendo que para as bolsas parciais, o custo deverá ser lançado com o valor parcial, o restante do custo deste aluno, será lançado na conta dos alunos não bolsistas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302010700,4.01.03.02.01.07.00,Outros Custos, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302020100,4.01.03.02.02.01.00,Custo dos Serviços Prestados a Pacientes Particulares,Contas que registram o custo da prestação do serviço para os pacientes particulares.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302020200,4.01.03.02.02.02.00,Custo dos Serviços Prestados a Convênios SUS,Contas que registram o custo da prestação do serviço para os pacientes atendidos através do convênio do SUS.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302020300,4.01.03.02.02.03.00,Custo dos Serviços Prestados a Convênios/Contratos/Parcerias,"Contas que registram o custo da prestação do serviço para os pacientes vinculados aos convênios/contratos/parcerias, exceto àqueles que estão no SUS.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302020400,4.01.03.02.02.04.00,Custo dos Serviços Prestados a Doações/Subvenções Vinculadas,"Contas que registram o custo da prestação do serviço para os pacientes vinculados à doações/subvenções vinculadas (Dec. 2.536/1998, art. 3, inciso V),  com destinação à área de saúde, preferencialmente segregadas por níveis federal, estadual e municipal.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302020500,4.01.03.02.02.05.00,Custo dos Serviços Prestados a Doações,"Contas que registram o custo da prestação do serviço para os pacientes vinculados às demais doações,  com destinação à área de saúde, exceto àquelas doações vinculadas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302020600,4.01.03.02.02.06.00,Custo dos Serviços Prestados a Gratuidade,"Contas que registram o custo da prestação do serviço para os pacientes com gratuidades do pagamento, exceto às vinculadas ao SUS.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302020700,4.01.03.02.02.07.00,Outros Custos, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302030100,4.01.03.02.03.01.00,Custo dos Serviços Prestados a Pacientes Particulares,Contas que registram o custo da prestação do serviço para os usuários particulares.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302030200,4.01.03.02.03.02.00,Custo dos Serviços Prestados a Convênios/Contratos/Parcerias,"Contas que registram o custo da prestação do serviço para os usuários vinculados aos convênios/contratos/parcerias, exceto àqueles que estão vinculados por doações e por subvenções.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302030300,4.01.03.02.03.03.00,Custo dos Serviços Prestados a Doações/Subvenções Vinculadas,"Contas que registram o custo da prestação do serviço para os usuários vinculados a doações/subvenções vinculadas (Dec. 2.536/1998, art. 3, inciso V),  com destinação à área de assistência social, preferencialmente segregadas por níveis federal, estadual e municipal.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302030400,4.01.03.02.03.04.00,Custo dos Serviços Prestados a Doações,"Contas que registram o custo da prestação do serviço para os pacientes vinculados às demais doações,  com destinação à área de saúde, exceto àquelas doações vinculadas.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302030500,4.01.03.02.03.05.00,Custo dos Serviços Prestados a Gratuidade,"Contas que registram o custo da prestação do serviço para os usuários com gratuidades do pagamento, exceto às atividades vinculadas por doações e por subvenções. Em especial, ao publico alvo da política nacional de assistência social.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302030600,4.01.03.02.03.06.00,Outros Custos, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302040100,4.01.03.02.04.01.00,Custo dos Serviços Prestados em Geral,"Contas que registram o custo da prestação do serviço para as demais atividades, não informadas anteriormente. ",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010302040200,4.01.03.02.04.02.00,Outros Custos, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501010100,4.01.05.01.01.01.00,Variações Cambiais Ativas,"Contas que registram os ganhos apurados em razão de variações ativas decorrentes da atualização dos direitos de crédito e obrigações, calculados com base nas variações das taxas de câmbio. 
Atenção:
1) as variações cambiais ativas decorrentes dos direitos de crédito e de obrigações, em função da taxa de câmbio, são consideradas como receita financeira, inclusive para fins de cálculo do lucro da 
exploração (Lei no 9.718, art. 9o c/c art. 17);
2) nas atividades de compra e venda, loteamento, incorporação e construção de imóveis, as variações cambiais ativas são reconhecidas como receita segundo as normas constantes da IN SRF no 84/79, de
20 de dezembro de 1979, da IN SRF no 23/83, de 25 de março de 1983, e da IN SRF no 67/88, de 21 de abril de 1988 (IN SRF no 25/99, de 25 de fevereiro de 1999).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501010200,4.01.05.01.01.02.00,"Ganhos Auferidos no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório dos ganhos auferidos, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País; 
b) os ganhos auferidos nas alienações, fora de bolsa, de ouro, ativo financeiro, e de participações societárias, exceto as alienações de participações societárias permanentes em sociedades coligadas e 
controladas e de participações societárias que permanecerem no ativo da pessoa jurídica até o término do ano-calendário seguinte ao de suas aquisições; e 
c) os rendimentos auferidos em operações de swap e no resgate de quota de fundo de investimento cujas carteiras sejam constituídas, no mínimo, por 67% (sessenta e sete por cento) de ações no mercado à vista de bolsa de valores ou entidade assemelhada (Lei no 9.532, de 1997, art. 28, alterado pela MP no 1.636, de 1998, art. 2o, e reedições).
Considera-se ganho o resultado positivo auferido nas operações citadas acima, realizadas em cada mês, admitida a dedução dos custos e despesas incorridos, necessários à realização das operações.
Atenção:
1) os ganhos auferidos em operações day-trade devem ser informados em conta específica;
2) o valor correspondente às perdas incorridas no mercado de renda variável, exceto day-trade, deve ser informado em conta específica; 
3) são consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros, as entidades cujo objeto social seja análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501010300,4.01.05.01.01.03.00,Ganhos em Operações Day-Trade,"Contas que registram os ganhos diários auferidos, em cada mês do período de apuração, em operações day-trade. Considera-se ganho o resultado positivo auferido nas operações citadas acima, realizadas em cada mês, admitida a dedução dos custos e despesas incorridos, necessários à realização das operações.Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado a vista, no mesmo dia.Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia.
Atenção: o valor correspondente às perdas incorridas nas operações day-trade deve ser informado em conta específica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501010400,4.01.05.01.01.04.00,Outras Receitas de Aplicações Financeiras,"Contas que registram receitas auferidas no período de apuração relativas a juros, descontos, lucro na operação de reporte, prêmio de resgate de títulos ou debêntures e rendimento nominal auferido em aplicações financeiras de renda fixa, não incluídas em outras contas. As receitas dessa natureza, derivadas de operações com títulos vencíveis após o encerramento do período de apuração, serão rateadas segundo o regime de competência.
Atenção:
1) as variações monetárias ativas decorrentes da atualização dos direitos de crédito e das obrigações, em função de índices ou coeficientes aplicáveis por disposição legal ou contratual, devem ser informadas como receita financeira;
2) as variações cambiais ativas devem ser informadas na conta específica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501010500,4.01.05.01.01.05.00,Ganhos na Alienação de Participações Não Integrantes do Ativo Permanente,"Contas que registram os ganhos auferidos na alienação de ações, títulos ou quotas de capital não integrantes do ativo permanente, desde que não incluídos em outra conta específica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501010600,4.01.05.01.01.06.00,Resultados Positivos em Participações Societárias,"Contas que registram:
a) os lucros e dividendos derivados de investimentos avaliados pelo custo de aquisição;
b) os ganhos por ajustes no valor de investimentos relevantes avaliados pelo método da equivalência patrimonial, decorrentes de lucros apurados nas controladas e coligadas;
Atenção: considera-se controlada a filial, a agência, a sucursal, a dependência ou o escritório de representação no exterior, sempre que os respectivos ativos e account.data_account_type_current_liabilitiess não estejam incluídos na contabilidade da investidora, por força de normatização específica.
c) as amortizações de deságios nas aquisições de investimentos avaliados pelo patrimônio líquido. O valor amortizado que for excluído do lucro líquido para determinação do lucro real deve ser controlado na Parte B do Livro de Apuração do Lucro Real até a alienação ou baixa da participação societária, quando, então, deve ser adicionado ao lucro líquido para determinação do lucro real no período de apuração em que for computado o ganho ou perda de capital havido.
d) as bonificações recebidas; 
Atenção:
1) as bonificações recebidas, decorrentes da incorporação de lucros ou reservas não tributados na forma do art. 35 da Lei no 7.713, de 1988, ou apurados nos anos-calendário de 1994 ou 1995, são 
consideradas a custo zero, não afetando o valor do investimento nem o resultado do período de apuração (art. 3o da Lei no 8.849, de 1994, e art. 3o da Lei no 9.064, de 1995); 
2) no caso de investimento avaliado pelo custo de aquisição, as bonificações recebidas, decorrentes da incorporação de lucros ou reservas tributados na forma do art. 35 da Lei no 7.713, de 1988, e de lucros ou reservas apurados no ano-calendário de 1993 ou a partir do ano-calendário de 1996, são registradas tomando-se como custo o valor da parcela dos lucros ou reservas capitalizados.
e) os lucros e dividendos de participações societárias avaliadas pelo custo de aquisição;
Atenção: os lucros ou dividendos recebidos em decorrência de participações societárias avaliadas pelo custo de aquisição adquiridas até 6 (seis) meses antes da data do recebimento devem ser registrados como diminuição do valor do custo, não sendo incluídos nesta conta.
f) os resultados positivos decorrentes de participações societárias no exterior avaliadas pelo patrimônio líquido, os dividendos de participações avaliadas pelo custo de aquisição e os resultados de equivalência patrimonial relativos a filiais, sucursais ou agências da pessoa jurídica localizadas no exterior, em decorrência de operações realizadas naquelas filiais, sucursais ou agências.Os lucros auferidos no exterior serão adicionados ao lucro líquido, para efeito de determinação do lucro real, no período de apuração correspondente ao balanço levantado em 31 de dezembro do ano-calendário em que tiverem sido disponibilizados, observando-se o disposto nos arts. 394 e 395 do Decreto no 3.000, de 1999, e no art. 74 da Medida Provisória no 2.158-35, de 24 de agosto de 2001.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501010700,4.01.05.01.01.07.00,Rendimentos e Ganhos de Capital Auferidos no Exterior,"Contas que registram os rendimentos e ganhos de capital auferidos no exterior diretamente pela pessoa jurídica domiciliada no Brasil, pelos seus valores antes de descontado o tributo pago no país de origem. 
Atenção:Os ganhos de capital referentes a alienações de bens e direitos do ativo permanente situados no exterior devem ser informados na conta Outras Receitas Não Operacionais ",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501010800,4.01.05.01.01.08.00,Reversão dos Saldos das Provisões Operacionais,Contas que registram a reversão de  saldos não utilizados das provisões constituídas no balanço do período de apuração imediatamente anterior.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501010900,4.01.05.01.01.09.00,Outras Receitas Operacionais,"Contas que registram todas as demais receitas que, por definição legal, sejam consideradas operacionais, tais como:
a) aluguéis de bens por empresa que não tenha por objeto a locação de móveis e imóveis;
b) recuperações de despesas operacionais de períodos de apuração anteriores, tais como: prêmios de seguros, importâncias levantadas das contas vinculadas do FGTS, ressarcimento de desfalques, roubos e furtos, etc. As recuperações de custos e despesas no decurso do próprio período de apuração devem ser creditadas diretamente às contas de resultado em que foram debitadas;
c) os créditos presumidos do IPI para ressarcimento do valor da Contribuição ao PIS/Pasep e Cofins;
d) multas ou vantagens a título de indenização em virtude de rescisão contratual (Lei no 9.430, de 1996, art. 70, § 3o, II);
e) o crédito presumido da contribuição para o PIS/Pasep e da Cofins concedido na 
forma do art. 3o da Lei no 10.147, de 2000.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010501011000,4.01.05.01.01.10.00,Outras, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107010100,4.01.07.01.01.00,Remunerações a Empregados,"Contas que registram os valores lançados como salários, gratificações, horas extras, adicionais e similares pag0s a empregados da entidade.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107010200,4.01.07.01.02.00,Indenizações Trabalhistas,"Contas que registram os valores lançados como abonos pecuniários, indenização de 40% do FGTS, indenizações determinadas pelo Juiz e similares pagas aos empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107010300,4.01.07.01.03.00,Remuneração a Dirigentes e a Conselho de Administração/Fiscal,"Contas que registram a despesa incorrida relativa à remuneração mensal e fixa atribuída ao titular de firma individual, aos sócios, diretores e administradores de sociedades, ou aos representantes legais de sociedades estrangeiras, as despesas incorridas com os salários indiretos concedidos pela empresa a administradores, diretores, gerentes e seus assessores (PN Cosit no 11, de 1992), e o valor referente às remunerações atribuídas aos membros do conselho fiscal/administração/consultivo.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107010400,4.01.07.01.04.00,Prestação de Serviços por Pessoa Física sem Vínculo Empregatício,"Contas que registram as despesas correspondentes aos serviços prestados por pessoa física que não tenha vínculo empregatício com a pessoa jurídica declarante, tais como: comissões, corretagens, gratificações, honorários e outras remunerações, inclusive as relativas a empreitadas de obras exclusivamente de trabalho e as decorrentes de fretes e carretos em geral.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107010500,4.01.07.01.05.00,Prestação de Serviço por Pessoa Jurídica,"Contas que registram, salvo se houver conta mais específica, o valor das despesas correspondentes aos serviços prestados por outra pessoa jurídica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107010600,4.01.07.01.06.00,Doações e Patrocínios de Caráter Cultural e Artístico (Lei no 8.313/1991),"Contas que registram as doações e patrocínios efetuados no período de apuração em favor de projetos culturais previamente aprovados pelo Ministério da Cultura ou pela Agência Nacional do Cinema (Ancine), observada a legislação de concessão dos projetos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107010700,4.01.07.01.07.00,"Doações a Instituições de Ensino e Pesquisa (Lei no 9.249/1995, art.13, § 2o)","Contas que registram as doações a instituições de ensino e pesquisa cuja criação tenha sido autorizada por lei federal e que preencham os requisitos dos incisos I e II do art. 213 da Constituição Federal, de 1988, que são:
a) comprovação de finalidade não-lucrativa e aplicação dos excedentes financeiros em educação;
b) assegurar a destinação do seu patrimônio a outra escola comunitária, filantrópica ou confessional, ou ao Poder Público, no caso de encerramento de suas atividades.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107010800,4.01.07.01.08.00,Doações a Entidades Civis,"Contas que registram as doações efetuadas a:
a) entidades civis, legalmente constituídas no Brasil, sem fins lucrativos, que prestem serviços gratuitos em benefício de empregados da pessoa jurídica doadora, e respectivos dependentes, ou em benefício da comunidade na qual atuem; e
b) Organizações da Sociedade Civil de Interesse Público (OSCIP), qualificadas segundo as normas estabelecidas na Lei no 9.790, de 23 de março de 1999.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107010900,4.01.07.01.09.00,Outras Contribuições e Doações,"Contas que registram as doações feitas, entre outras, aos Fundos controlados pelos Conselhos Municipais, Estaduais e Nacional dos Direitos da Criança e do Adolescente.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011000,4.01.07.01.10.00,FGTS (sem indenização 40%),"Contas que registram o FGTS, inclusive os valores do FGTS do 13º. salário. Não informar os valores de indenização da multa de 40% do FGTS nesse item, e sim, na conta Indenizações Trabalhistas .",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011100,4.01.07.01.11.00,"Assistência Médica, Odontológica, Medicamentos, Aparelhos Ortopédicos e Similares","Contas que registram as despesas com assistência médica, odontológica e farmacêutica. 
Atenção: o valor referente à contratação de serviços de profissionais liberais sem vínculo empregatício ou de sociedades civis deve ser informado nas contas Prestação de Serviços por Pessoa Física sem Vínculo Empregatício  ou Prestação de Serviço por Pessoa Jurídica , conforme o caso.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011200,4.01.07.01.12.00,Provisões para Férias e 13o Salário de Empregados,"Contas que registram as despesas com a constituição de provisões para: a) pagamento de remuneração correspondente a férias e adicional de férias de empregados, inclusive encargos sociais (Decreto no 3.000, de 1999, art. 337, e PN CST no 7, de 1980); b) o 13o salário, inclusive encargos sociais (Decreto no 3.000, de 1999, art. 338).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011300,4.01.07.01.13.00,Demais Provisões,Contas que registram as despesas com provisões não relacionadas nas contas específicas.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011400,4.01.07.01.14.00,Arrendamento Mercantil,"Contas que registram as despesas, não computadas nos custos, pagas ou creditadas a título de contraprestação de arrendamento mercantil, decorrentes de contrato celebrado com observância da Lei no 6.099, de 12 de setembro de 1974, com as alterações da Lei no 7.132, de 26 de outubro de 1983, e da Portaria MF no 140, de 1984",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011500,4.01.07.01.15.00,Aluguéis,Contas que registram as despesas com aluguéis não decorrentes de arrendamento mercantil.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011600,4.01.07.01.16.00,Despesas com Veículos e de Conservação de Bens e Instalações,"Contas que registram as despesas relativas aos bens que não estejam ligados diretamente à produção, as realizadas com reparos que não impliquem aumento superior a um ano da vida útil do bem, prevista no ato de sua aquisição, e as relativas a combustíveis e lubrificantes para veículos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011700,4.01.07.01.17.00,Propaganda e Publicidade,"Contas que registram as despesas com propaganda e publicidade. 
Atenção: o valor referente à contratação de serviços de profissionais liberais sem vínculo empregatício ou de sociedades civis deve ser informado nas contas Prestação de Serviços por Pessoa Física sem Vínculo Empregatício  ou Prestação de Serviço por Pessoa Jurídica , conforme o caso.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011800,4.01.07.01.18.00,Multas,Contas que registram as despesas com multas.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107011900,4.01.07.01.19.00,Encargos de Depreciação e Amortização,"Contas que registram apenas os encargos a esses títulos, com bens não aplicados diretamente na produção. Inclui a amortização dos ajustes de variação cambial contabilizada no ativo diferido, relativa à atividade geral da pessoa jurídica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107012000,4.01.07.01.20.00,Repasses para Outras Entidades (Sindicatos/Federações/Confederações),Contas que foram repassadas parte das contribuições/doações/mensalidades e similares para Sindicatos/Federações/Confederações.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107012100,4.01.07.01.21.00,Contribuições Previdenciárias Patronais,"Contas que registram as contribuições previdenciárias devidas. No caso de imunes/isentas, informar o valor da contribuição previdenciária patronal devida como se sem isenção estivesse, devendo fazer um novo lançamento de reversão para evidenciar que é isenta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107012200,4.01.07.01.22.00,COFINS,"Contas que registram a Cofins devida. No  caso de imunes/isentas, informar o valor da Cofins devida como se sem isenção estivesse, devendo fazer um novo lançamento de reversão para evidenciar que é isenta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107012300,4.01.07.01.23.00,CSLL,"Contas que registram a CSLL devida. No caso de imunes/isentas, informar o valor da CSLL devida como se sem isenção estivesse, devendo fazer um novo lançamento de reversão para evidenciar que é isenta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107012400,4.01.07.01.24.00,PIS/PASEP,Contas que registram o valor da contribuição para o PIS/PASEP devida.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107012500,4.01.07.01.25.00,CPMF,Contas que registram o valor da CPMF devida.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107012600,4.01.07.01.26.00,"Demais Impostos, Taxas e Contribuições, exceto as citadas acima.","Contas que registram os demais impostos, taxas e contribuições, exceto: 
a) incorporadas ao custo de bens do ativo permanente;
b) correspondentes aos impostos não recuperáveis, incorporados ao custo das matérias-primas, materiais secundários, materiais de embalagem e mercadorias destinadas à revenda;
c) correspondentes aos impostos recuperáveis;
d) correspondentes aos impostos e contribuições redutores da receita bruta.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_40107012700,4.01.07.01.27.00,Outras Despesas Operacionais, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010901010100,4.01.09.01.01.01.00,(-) Variações Cambiais Passivas,"Contas que registram as perdas monetárias passivas resultantes da atualização dos direitos de créditos e das obrigações, calculadas com base nas variações nas taxas de câmbio (Lei no 9.069, de 1995, art.52, e Lei no 9.249, de 1995, art. 8o), inclusive a variação cambial passiva correspondente: 
a) à atualização das obrigações e dos créditos em moeda estrangeira, registrada em qualquer data e apurada no encerramento do período de apuração em função da taxa de câmbio vigente;
b) às operações com moeda estrangeira e conversão de obrigações para moeda nacional, ou novação dessas obrigações, ou sua extinção, total ou parcial, em virtude de capitalização, dação em pagamento, compensação, ou qualquer outro modo, desde que observadas as condições fixadas pelo Banco Central do Brasil.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010901010200,4.01.09.01.01.02.00,"(-) Perdas Incorridas no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório das perdas incorridas, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País;
b) as perdas incorridas nas alienações, fora de bolsa, de ouro, ativo financeiro e de participações societárias, exceto as alienações de participações societárias permanentes em sociedades  coligadas e controladas e de participações societárias, que permanecerem no ativo da pessoa jurídica até o término do ano-calendário seguinte ao de suas aquisições; e
c) as perdas em operações de swap e no resgate de quota de fundo de investimento que mantenha, no mínimo, 67% (sessenta e sete por cento) de ações negociadas no mercado a vista de bolsa de valores ou entidade assemelhada (Lei no 9.532, de 1997, art. 28, alterado pela MP no 1.636, de 1998, art. 2o, e reedições).São consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros, as entidades cujo objeto social seja análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM). Atenção: As perdas apuradas em operações day-trade devem ser informadas em conta própria.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010901010300,4.01.09.01.01.03.00,(-) Perdas em Operações Day-Trade,"Contas que registram o somatório das perdas diárias apuradas, em cada mês do período de apuração, em operações day-trade.Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado a vista, no mesmo dia.Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010901010400,4.01.09.01.01.04.00,(-) Outras Despesas de Aplicações,"Contas que registram as despesas relativas a juros, não incluídas  em outras contas, a descontos de títulos de crédito e outros títulos. Tais despesas serão obrigatoriamente rateadas, segundo o regime de competência.
Atenção: 
1) as variações monetárias passivas decorrentes da atualização das 
obrigações, em função de índices ou coeficientes aplicáveis por disposição legal ou contratual, devem ser informadas como despesas financeiras;
2) as variações cambiais passivas não devem ser informadas nesta conta, e sim na conta Variações Cambiais Passivas .",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010901010500,4.01.09.01.01.05.00,(-) Prejuízos na Alienação de Participações Não Integrantes do Ativo Permanente,"Contas que registram os prejuízos havidos em virtude de alienação, títulos não integrantes do ativo permanente, desde que não incluídos nas contas acima.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010901010600,4.01.09.01.01.06.00,(-) Resultados Negativos em Participações Societárias,"Contas que registram as perdas por ajustes no valor de investimentos relevantes, avaliados pelo método da equivalência patrimonial, decorrentes de prejuízos apurados nas controladas e coligadas. 
Atenção: considera-se controlada a filial, a agência, a sucursal, a dependência ou o escritório de representação no exterior, sempre que os respectivos ativos e account.data_account_type_current_liabilitiess não estejam incluídos na contabilidade da investidora, por força de normatização específica.
Devem, também, ser indicados nesta conta os resultados negativos derivados de participações societárias no exterior, avaliadas pelo patrimônio líquido. Incluem-se, nestas informações, as perdas apuradas em filiais, sucursais e agências da pessoa jurídica localizadas no exterior.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010901010700,4.01.09.01.01.07.00,(-) Perdas em Operações Realizadas no Exterior,"Contas que registram as perdas em operações realizadas no exterior diretamente pela pessoa jurídica domiciliada no Brasil, com exceção das perdas de capital decorrentes da alienação de bens e direitos do ativo permanente situados no exterior, que devem ser indicadas na conta Outras Despesas Não Operacionais.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4010901010800,4.01.09.01.01.08.00,Outras Despesas Operacionais, ,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4030101010100,4.03.01.01.01.01.00,Receitas de Alienações de Bens e Direitos do Ativo Permanente.,"Contas que registram as receitas auferidas por meio de alienações, inclusive por desapropriação de bens e direitos do ativo permanente. O valor relativo às receitas obtidas pela venda de sucata e de bens ou direitos do ativo permanente baixados em virtude de terem se tornado imprestáveis, obsoletos ou caído em desuso deve ser informado na conta Outras Receitas Não Operacionais . Os valores correspondentes ao ganho ou perda de capital decorrente da alienação de bens e direitos do ativo permanente situados no exterior devem ser indicados, pelo seu resultado, nas contas Outras Receitas Não Operacionais  ou Outras Despesas Não Operacionais , conforme o caso.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4030101010200,4.03.01.01.01.02.00,Outras Receitas Não Operacionais,"Contas que registram:
a) todas as demais receitas decorrentes de operações não incluídas nas atividades principais e acessórias da empresa, tais como: a reversão do saldo da provisão para perdas prováveis na realização de investimentos e a reserva de reavaliação realizada no período de apuração, quando computada em conta de resultado; 
b) os ganhos de capital por variação na percentagem de participação no capital social de coligada ou controlada, quando o investimento for avaliado pela 
equivalência patrimonial (Decreto no 3.000, de 1999, art. 428);
c) os ganhos de capital decorrentes da alienação de bens e direitos do ativo permanente situados no exterior. 
Devem ser indicadas tanto as contas que registram as receitas quanto as que registram os custos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4030201010100,4.03.02.01.01.01.00,(-) Valor Contábil dos Bens e Direitos Alienados,"Contas que registram o valor contábil dos bens do ativo permanente baixados no curso do período de apuração, cuja receita da venda tenha sido indicada na conta Receitas de Alienações de Bens e Direitos do Ativo Permanente . O valor contábil de bens ou direitos baixados em virtude de terem se tornado imprestáveis, obsoletos ou caído em desuso e o valor contábil de bens ou direitos situados no exterior devem ser informados na conta Outras Receitas Não Operacionais.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_4030201010200,4.03.02.01.01.02.00,(-) Outras Despesas Não Operacionais,"Contas que registram:
a) o valor contábil dos bens do ativo permanente baixados no curso do período de apuração não incluídos na conta precedente e a despesa com a constituição da provisão para perdas prováveis na realização de investimentos; 
Atenção: sobre a definição de valor contábil, consultar o § 1o do art. 418 e o art. 426, ambos do Decreto no 3.000, de 1999.
b) as perdas de capital por variação na percentagem de participação no capital social de coligada ou controlada no Brasil, quando o investimento for avaliado pela equivalência patrimonial (Decreto no 3.000, de 1999, art. 428).",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010100,5.01.01.01.00,Consumo de Insumos,"Contas que registram o consumo, durante o período de apuração, de matéria-prima, material direto e material de embalagem, no mercado interno e externo, para utilização no processo produtivo, os valores referentes aos custos com transporte e seguro até o estabelecimento do contribuinte, os tributos não recuperáveis devidos na importação e o custo relativo ao desembaraço aduaneiro.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010400,5.01.01.04.00,Remuneração a Dirigentes de Ligados à Produção,"Contas que registram:
a) a remuneração mensal e fixa dos dirigentes diretamente ligados à produção, pelo valor total do custo incorrido no período de apuração, exceto os encargos sociais (Previdência Social e FGTS) que são informados em conta distinta;
b) o valor relativo aos custos incorridos com salários indiretos concedidos pela empresa a administradores, diretores, gerentes e seus assessores, se ligados diretamente à produção (PN Cosit nº 11, de 30 de setembro de 1992). 
Atenção: deve ser incluído nesta conta o valor das gratificações dos dirigentes ligados à produção, inclusive o 13º salário.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010500,5.01.01.05.00,Custo do Pessoal Aplicado na Produção,"Contas que representem do custo com ordenados, salários e outros custos com empregados ligados à produção da empresa, tais como: moradia, seguro de vida e outras de caráter remuneratório. Inclusive os custos com supervisão direta, manutenção e guarda das instalações, decorrentes de vínculo empregatício com a pessoa jurídica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010503,5.01.01.05.03,Planos de Poupança e Investimentos de Empregados Ligados à Produção,"Contas que registram o valor total dos gastos efetuados com Planos de Poupança e Investimentos (PAIT), relativos ao pessoal ligado à produção",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010505,5.01.01.05.05,Fundo de Aposentadoria Programada Individual de Empregados Ligados à Produção,"Contas que registram o valor total dos gastos efetuados com Fundos de Aposentadoria Programada Individual (FAPI), relativos ao pessoal ligado à produção",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010507,5.01.01.05.07,Plano de Previdência Privada de Empregados Ligados à Produção,"Contas que registram o valor total dos gastos efetuados com Planos de Previdência Privada, relativos ao pessoal ligado à produção.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010509,5.01.01.05.09,Outros Gastos com Pessoal Ligado à Produção,"Contas que registram os gastos com empregados, computados nos custos, não enquadrados nas contas precedentes.
Atenção:  não deve ser informado nesta conta o valor referente às participações dos empregados no lucro da pessoa jurídica. Esse valor deve ser informado na conta Participações de Empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010600,5.01.01.06.00,Prestação de Serviços por Pessoa Física sem Vínculo Empregatício,"Contas que registram, salvo se houver conta mais específica, os gastos correspondentes aos serviços prestados por pessoa física que não tenha vínculo empregatício com a pessoa jurídica, tais como: comissões, corretagens, gratificações, honorários, direitos autorais e outras remunerações, inclusive as relativas a empreitadas de obras exclusivamente de trabalho e as decorrentes de fretes e carretos em gera, computadas nos custos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010700,5.01.01.07.00,Prestação de Serviço Pessoa Jurídica,"Contas que registram, salvo se houver conta mais específica, o valor dos gastos correspondentes aos serviços prestados por outra pessoa jurídica à pessoa jurídica declarante, computados nos custos",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010800,5.01.01.08.00,Serviços Prestados por Cooperativa de Trabalho,Contas que registram os serviços prestados por cooperativa de trabalho,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501010900,5.01.01.09.00,Locação de Mão-de-obra,"Contas que registram o valor total dos gastos efetuados no período com a contratação de serviços executados mediante cessão de mão-de-obra ou empreitada, inclusive em regime temporário, sujeitos à retenção de contribuição previdenciária, nos termos do art. 219 do Regulamento da Previdência Social - RPS, aprovado pelo Decreto nº 3.048, de 1999",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011000,5.01.01.10.00,Encargos Sociais – Previdência Social,"Contas que registram as contribuições para a Previdência Social (inclusive dos dirigentes de indústria - PN CST no 35, de 31 de agosto de 1981), relativas ao pessoal ligado diretamente à produção.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011100,5.01.01.11.00,Encargos Sociais – FGTS,"Contas que registram as contribuições para o FGTS (inclusive dos dirigentes de indústria - PN CST no 35, de 31 de agosto de 1981), relativas ao pessoal ligado diretamente à produção.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011200,5.01.01.12.00,Encargos Sociais – Outros,"Contas que registram encargos sociais, relativos ao pessoal ligado diretamente à produção, não classificados nas contas Encargos Sociais – Previdência Social ou Encargos Sociais – FGTS.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011300,5.01.01.13.00,Alimentação do Trabalhador,"Contas que registram os custos com alimentação do pessoal ligado diretamente à produção, realizados durante o período de apuração, ainda que a pessoa jurídica não tenha Programa de Alimentação do Trabalhador aprovado pelo Ministério do Trabalho.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011400,5.01.01.14.00,Manutenção e Reparo de Bens Aplicados na Produção,Contas que representam somente os custos realizados com reparos que não implicaram aumento superior a um ano da vida útil prevista no ato da aquisição do bem.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011500,5.01.01.15.00,Arrendamento Mercantil,"Contas que representam o valor do custo incorrido a título de contraprestação de arrendamento mercantil de bens alocados na produção, segundo contratos celebrados com observância da Lei nº 6.099, de 12 de setembro de 1974, com as alterações da Lei nº 7.132, de 26 de outubro de 1983. Os custos com aluguel de outros bens alocados à produção, mediante contrato diferente do de arrendamento mercantil, devem ser indicados em ""Outros Custos"". Os valores referentes a bens que não sejam intrinsecamente relacionados com a produção devem ser informados na conta Arrendamento Mercantil do grupo DESPESAS OPERACIONAIS DAS ATIVIDADES EM GERAL",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011600,5.01.01.16.00,"Encargos de Depreciação, Amortização e Exaustão",Contas que registram os encargos a esses títulos com bens aplicados diretamente na produção. Os encargos que não forem decorrentes de bens intrinsecamente relacionados com a produção devem ser informados na conta Encargos de Depreciação e Amortização do grupo DESPESAS OPERACIONAIS DAS ATIVIDADES EM GERAL.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011700,5.01.01.17.00,Constituição de Provisões,Contas que registram os encargos com a constituição de provisões que devam ser imputados aos custos de produção da empresa no período de apuração.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011800,5.01.01.18.00,Serviços Prestados por Pessoa Física sem Vínculo Empregatício,"Contas que registram, salvo se houver conta mais específica neste plano referencial, os custos correspondentes aos serviços prestados à pessoa jurídica por pessoa física sem vínculo empregatício, relacionados com a atividade industrial da pessoa jurídica .",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501011900,5.01.01.19.00,Serviços Prestados Pessoa Jurídica,"Contas que registram, salvo se houver conta mais específica neste plano referencial, os custos correspondentes aos serviços prestados por pessoa jurídica, relacionados com atividade industrial da pessoa jurídica declarante.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501012000,5.01.01.20.00,Royalties e Assistência Técnica – PAÍS,"Contas que registram  as importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no Brasil, a título de royalties e assistência técnica, científica ou assemelhada, que estejam relacionadas com a atividade industrial.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501012100,5.01.01.21.00,Royalties e Assistência Técnica – EXTERIOR,"Contas que registram as importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no exterior, a título de royalties e assistência técnica, científica ou assemelhada, que estejam relacionadas com a atividade industrial.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501019000,5.01.01.90.00,Outros Custos,"Contas que representam os demais custos da empresa no processo de produção, para os quais não haja conta maIs específica ou cujas classificações contábeis não se adaptem à nomenclatura específica, tais como: custo referente ao valor de bens de consumo eventual; as quebras ou perdas de estoque, e as ocorridas na fabricação, no transporte e manuseio.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030200,5.01.03.02.00,Material Aplicado na Produção de Serviços,Contas correspondentes aos materiais aplicados diretamente na produção de serviços dos serviços durante o período de apuração.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030400,5.01.03.04.00,Remuneração a Dirigentes ligados à Produção de Serviços,"Contas que registram:
a) a remuneração mensal e fixa dos dirigentes diretamente ligados à produção de serviços, pelo valor total do custo incorrido no período de apuração, exceto os encargos sociais (Previdência Social e FGTS) que são informados em conta distinta;
b) o valor relativo aos custos incorridos com salários indiretos concedidos pela empresa a administradores, diretores, gerentes e seus assessores, se ligados diretamente à produção de serviços (PN Cosit nº 11, de 30 de setembro de 1992).
Atenção: deve ser incluído nesta conta o valor das gratificações dos dirigentes ligados à produção de serviços, inclusive o 13º salário.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030500,5.01.03.05.00,Custo do Pessoal Aplicado na Produção de Serviços,"Contas que representem do custo com ordenados, salários e outros custos com empregados ligados à produção de serviços da empresa, tais como: moradia, seguro de vida e outras de caráter remuneratório. Inclusive os custos com supervisão direta, manutenção e guarda das instalações, decorrentes de vínculo empregatício com a pessoa jurídica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030503,5.01.03.05.03,Planos de Poupança e Investimentos de Empregados Ligados à Produção de Serviços,"Contas que registram o valor total dos gastos efetuados com Planos de Poupança e Investimentos (PAIT), relativos ao pessoal ligado à produção de serviços",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030505,5.01.03.05.05,Fundo de Aposentadoria Programada Individual de Empregados Ligados à Produção de Serviços,"Contas que registram o valor total dos gastos efetuados com Fundos de Aposentadoria Programada Individual (FAPI), relativos ao pessoal ligado à produção de serviços",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030507,5.01.03.05.07,Plano de Previdência Privada de Empregados Ligados à Produção de Serviços,"Contas que registram o valor total dos gastos efetuados com Planos de Previdência Privada, relativos ao pessoal ligado à produção de serviços.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030509,5.01.03.05.09,Outros Gastos com Pessoal Ligado à Produção de Serviços,"Contas que registram os gastos com empregados, computados nos custos, não enquadrados nas contas precedentes
Atenção:  não deve ser informado nesta conta o valor referente às participações dos empregados no lucro da pessoa jurídica. Esse valor deve ser informado na conta Participações de Empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030600,5.01.03.06.00,Prestação de Serviços por Pessoa Física sem Vínculo Empregatício,"Contas que registram, salvo se houver conta mais específica, os gastos correspondentes aos serviços prestados por pessoa física que não tenha vínculo empregatício com a pessoa jurídica, tais como: comissões, corretagens, gratificações, honorários, direitos autorais e outras remunerações, inclusive as relativas a empreitadas de obras exclusivamente de trabalho e as decorrentes de fretes e carretos em gera, computadas nos custos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030700,5.01.03.07.00,Prestação de Serviço Pessoa Jurídica,"Contas que registram, salvo se houver conta mais específica, o valor dos gastos correspondentes aos serviços prestados por outra pessoa jurídica à pessoa jurídica declarante, computados nos custos",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030800,5.01.03.08.00,Serviços Prestados por Cooperativa de Trabalho,Contas que registram os serviços prestados por cooperativa de trabalho,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501030900,5.01.03.09.00,Locação de Mão-de-obra,"Contas que registram o valor total dos gastos efetuados no período com a contratação de serviços executados mediante cessão de mão-de-obra ou empreitada, inclusive em regime temporário, sujeitos à retenção de contribuição previdenciária, nos termos do art. 219 do Regulamento da Previdência Social - RPS, aprovado pelo Decreto nº 3.048, de 1999",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031000,5.01.03.10.00,Encargos Sociais – Previdência Social,"Contas que registram as contribuições para a Previdência Social (inclusive dos dirigentes de indústria - PN CST no 35, de 31 de agosto de 1981), relativas ao pessoal ligado diretamente à produção de serviços.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031100,5.01.03.11.00,Encargos Sociais – FGTS,"Contas que registram as contribuições para o FGTS (inclusive dos dirigentes de indústria – PN CST no 35, de 31 de agosto de 1981), relativas ao pessoal ligado diretamente à produção de serviços.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031200,5.01.03.12.00,Encargos Sociais – Outros,"Contas que registram encargos sociais, relativos ao pessoal ligado diretamente à produção de serviços, não classificados nas contas Encargos Sociais - Previdência Social ou Encargos Sociais - FGTS.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031300,5.01.03.13.00,Alimentação do Trabalhador,"Contas que registram os custos com alimentação do pessoal ligado diretamente à produção de serviços, realizados durante o período de apuração, ainda que a pessoa jurídica não tenha Programa de Alimentação do Trabalhador aprovado pelo Ministério do Trabalho.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031400,5.01.03.14.00,Manutenção e Reparo de Bens Aplicados na Produção de Serviços,Contas que representam somente os custos realizados com reparos que não implicaram aumento superior a um ano da vida útil prevista no ato da aquisição do bem.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031500,5.01.03.15.00,Arrendamento Mercantil,"Contas que representam o valor do custo incorrido a título de contraprestação de arrendamento mercantil de bens alocados na produção de serviços, segundo contratos celebrados com observância da Lei nº 6.099, de 12 de setembro de 1974, com as alterações da Lei nº 7.132, de 26 de outubro de 1983. Os custos com aluguel de outros bens alocados à produção de serviços, mediante contrato diferente do de arrendamento mercantil, devem ser indicados em ""Outros Custos"". Os valores referentes a bens que não sejam intrinsecamente relacionados com a produção de serviços devem ser informados na conta Arrendamento Mercantil do grupo DESPESAS OPERACIONAIS DAS ATIVIDADES EM GERAL.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031600,5.01.03.16.00,"Encargos de Depreciação, Amortização e Exaustão",Contas que registram os encargos a esses títulos com bens aplicados diretamente na produção de serviços. Os encargos que não forem decorrentes de bens intrinsecamente relacionados com a produção de serviços devem ser informados na conta Encargos de Depreciação e Amortização do grupo DESPESAS OPERACIONAIS DAS ATIVIDADES EM GERAL.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031700,5.01.03.17.00,Constituição de Provisões,Contas que registram os encargos com a constituição de provisões que devam ser imputados aos custos de produção de serviços da empresa no período de apuração.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031800,5.01.03.18.00,Serviços Prestados por Pessoa Física sem Vínculo Empregatício,"Contas que registram, salvo se houver conta mais específica neste plano referencial, os custos correspondentes aos serviços prestados à pessoa jurídica por pessoa física sem vínculo empregatício, relacionados com a atividade industrial da pessoa jurídica .",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501031900,5.01.03.19.00,Serviços Prestados Pessoa Jurídica,"Contas que registram, salvo se houver conta mais específica neste plano referencial, os custos correspondentes aos serviços prestados por pessoa jurídica, relacionados com atividade industrial da pessoa jurídica declarante.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501032000,5.01.03.20.00,Royalties e Assistência Técnica – PAÍS,"Contas que registram as importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no Brasil, a título de royalties e assistência técnica, científica ou assemelhada, que estejam relacionadas com a atividade industrial.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501032100,5.01.03.21.00,Royalties e Assistência Técnica – EXTERIOR,"Contas que registram as importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no exterior, a título de royalties e assistência técnica, científica ou assemelhada, que estejam relacionadas com a atividade industrial.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501039000,5.01.03.90.00,Outros Custos,"Contas que representam os demais custos da empresa no processo de produção de serviços, para os quais não haja conta mais específica ou cujas classificações contábeis não se adaptem à nomenclatura específica, tais como: custo referente ao valor de bens de consumo eventual; as quebras ou perdas de estoque, e as ocorridas na fabricação, no transporte e manuseio.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050100,5.01.05.01.00,Consumo de Insumos,"Contas que registram o consumo, durante o período de apuração, de matéria-prima, material secundário e material de embalagem, no mercado interno e externo, para utilização no processo produtivo, os valores referentes aos custos com transporte e seguro até o estabelecimento do contribuinte, os tributos não recuperáveis devidos na importação e o custo relativo ao desembaraço aduaneiro.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050400,5.01.05.04.00,Remuneração a Dirigentes de Ligados à Produção,"Contas que registram:
a) a remuneração mensal e fixa dos dirigentes diretamente ligados à produção, pelo valor total do custo incorrido no período de apuração, exceto os encargos sociais (Previdência Social e FGTS) que são informados em conta distinta;
b) o valor relativo aos custos incorridos com salários indiretos concedidos pela empresa a administradores, diretores, gerentes e seus assessores, se ligados diretamente à produção (PN Cosit nº 11, de 30 de setembro de 1992). 
Atenção: deve ser incluído nesta conta o valor das gratificações dos dirigentes ligados à produção, inclusive o 13º salário.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050500,5.01.05.05.00,Custo do Pessoal Aplicado na Produção,"Contas que representem do custo com ordenados, salários e outros custos com empregados ligados à produção da empresa, tais como: moradia, seguro de vida e outras de caráter remuneratório. Inclusive os custos com supervisão direta, manutenção e guarda das instalações, decorrentes de vínculo empregatício com a pessoa jurídica.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050503,5.01.05.05.03,Planos de Poupança e Investimentos de Empregados Ligados à Produção,"Contas que registram o valor total dos gastos efetuados com Planos de Poupança e Investimentos (PAIT), relativos ao pessoal ligado à produção",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050505,5.01.05.05.05,Fundo de Aposentadoria Programada Individual de Empregados Ligados à Produção,"Contas que registram o valor total dos gastos efetuados com Fundos de Aposentadoria Programada Individual (FAPI), relativos ao pessoal ligado à produção",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050507,5.01.05.05.07,Plano de Previdência Privada de Empregados Ligados à Produção,"Contas que registram o valor total dos gastos efetuados com Planos de Previdência Privada, relativos ao pessoal ligado à produção.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050509,5.01.05.05.09,Outros Gastos com Pessoal Ligado à Produção,"Contas que registram os gastos com empregados, computados nos custos, não enquadrados nas contas precedentes.
Atenção:  não deve ser informado nesta conta o valor referente às participações dos empregados no lucro da pessoa jurídica. Esse valor deve ser informado na conta Participações de Empregados.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050600,5.01.05.06.00,Prestação de Serviços por Pessoa Física sem Vínculo Empregatício,"Contas que registram, salvo se houver conta mais específica, os gastos correspondentes aos serviços prestados por pessoa física que não tenha vínculo empregatício com a pessoa jurídica, tais como: comissões, corretagens, gratificações, honorários, direitos autorais e outras remunerações, inclusive as relativas a empreitadas de obras exclusivamente de trabalho e as decorrentes de fretes e carretos em gera, computadas nos custos.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050700,5.01.05.07.00,Prestação de Serviço Pessoa Jurídica,"Contas que registram, salvo se houver conta mais específica, o valor dos gastos correspondentes aos serviços prestados por outra pessoa jurídica à pessoa jurídica declarante, computados nos custos",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050800,5.01.05.08.00,Serviços Prestados por Cooperativa de Trabalho,Contas que registram os serviços prestados por cooperativa de trabalho,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501050900,5.01.05.09.00,Locação de Mão-de-obra,"Contas que registram o valor total dos gastos efetuados no período com a contratação de serviços executados mediante cessão de mão-de-obra ou empreitada, inclusive em regime temporário, sujeitos à retenção de contribuição previdenciária, nos termos do art. 219 do Regulamento da Previdência Social - RPS, aprovado pelo Decreto nº 3.048, de 1999",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051000,5.01.05.10.00,Encargos Sociais – Previdência Social,"Contas que registram as contribuições para a Previdência Social (inclusive dos dirigentes de indústria - PN CST no 35, de 31 de agosto de 1981), relativas ao pessoal ligado diretamente à produção.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051100,5.01.05.11.00,Encargos Sociais – FGTS,"Contas que registram as contribuições para o FGTS (inclusive dos dirigentes de indústria - PN CST no 35, de 31 de agosto de 1981), relativas ao pessoal ligado diretamente à produção.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051200,5.01.05.12.00,Encargos Sociais – Outros,"Contas que registram encargos sociais, relativos ao pessoal ligado diretamente à produção, não classificados nas contas Encargos Sociais – Previdência Social ou Encargos Sociais – FGTS.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051300,5.01.05.13.00,Alimentação do Trabalhador,"Contas que registram os custos com alimentação do pessoal ligado diretamente à produção, realizados durante o período de apuração, ainda que a pessoa jurídica não tenha Programa de Alimentação do Trabalhador aprovado pelo Ministério do Trabalho.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051400,5.01.05.14.00,Manutenção e Reparo de Bens Aplicados na Produção,Contas que representam somente os custos realizados com reparos que não implicaram aumento superior a um ano da vida útil prevista no ato da aquisição do bem.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051500,5.01.05.15.00,Arrendamento Mercantil,"Contas que representam o valor do custo incorrido a título de contraprestação de arrendamento mercantil de bens alocados na produção, segundo contratos celebrados com observância da Lei nº 6.099, de 12 de setembro de 1974, com as alterações da Lei nº 7.132, de 26 de outubro de 1983. Os custos com aluguel de outros bens alocados à produção, mediante contrato diferente do de arrendamento mercantil, devem ser indicados em ""Outros Custos"". Os valores referentes a bens que não sejam intrinsecamente relacionados com a produção devem ser informados na conta Arrendamento Mercantil do grupo DESPESAS OPERACIONAIS DA ATIVIDADE RURAL.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051600,5.01.05.16.00,"Encargos de Depreciação, Amortização e Exaustão",Contas que registram os encargos a esses títulos com bens aplicados diretamente na produção. Os encargos que não forem decorrentes de bens intrinsecamente relacionados com a produção devem ser informados na conta Encargos de Depreciação e Amortização do grupo DESPESAS OPERACIONAIS DA ATIVIDADE RURAL.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051700,5.01.05.17.00,Constituição de Provisões,Contas que registram os encargos com a constituição de provisões que devam ser imputados aos custos de produção da empresa no período de apuração.,account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051800,5.01.05.18.00,Serviços Prestados por Pessoa Física sem Vínculo Empregatício,"Contas que registram, salvo se houver conta mais específica neste plano referencial, os custos correspondentes aos serviços prestados à pessoa jurídica por pessoa física sem vínculo empregatício, relacionados com a atividade industrial da pessoa jurídica .",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501051900,5.01.05.19.00,Serviços Prestados Pessoa Jurídica,"Contas que registram, salvo se houver conta mais específica neste plano referencial, os custos correspondentes aos serviços prestados por pessoa jurídica, relacionados com atividade industrial da pessoa jurídica declarante.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501052000,5.01.05.20.00,Royalties e Assistência Técnica – PAÍS,"Contas que registram  as importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no Brasil, a título de royalties e assistência técnica, científica ou assemelhada, que estejam relacionadas com a atividade industrial.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501052100,5.01.05.21.00,Royalties e Assistência Técnica – EXTERIOR,"Contas que registram as importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no exterior, a título de royalties e assistência técnica, científica ou assemelhada, que estejam relacionadas com a atividade industrial.",account.data_account_type_expenses,,l10n_br_account_chart_template
account_template_501059000,5.01.05.90.00,Outros Custos,"Contas que representam os demais custos da empresa no processo de produção, para os quais não haja conta maIs específica ou cujas classificações contábeis não se adaptem à nomenclatura específica, tais como: custo referente ao valor de bens de consumo eventual; as quebras ou perdas de estoque, e as ocorridas na fabricação, no transporte e manuseio.",account.data_account_type_expenses,,l10n_br_account_chart_template

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_icms_0" model="account.tax.group">
            <field name="name">ICMS 0%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_7" model="account.tax.group">
            <field name="name">ICMS 7%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_12" model="account.tax.group">
            <field name="name">ICMS 12%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_19" model="account.tax.group">
            <field name="name">ICMS 19%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_26" model="account.tax.group">
            <field name="name">ICMS 26%</field>
            <field name="country_id" ref="base.br"/>
        </record>

        <record id="tax_group_irpj_0" model="account.tax.group">
            <field name="name">IRPJ 0%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_pis_0" model="account.tax.group">
            <field name="name">PIS 0%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_pis_065" model="account.tax.group">
            <field name="name">PIS 0.65%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_cofins_0" model="account.tax.group">
            <field name="name">COFINS 0%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_cofins_3" model="account.tax.group">
            <field name="name">COFINS 3%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ir_0" model="account.tax.group">
            <field name="name">IR 0%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_issqn_0" model="account.tax.group">
            <field name="name">ISSQN 0%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_issqn_1" model="account.tax.group">
            <field name="name">ISSQN 1%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_issqn_2" model="account.tax.group">
            <field name="name">ISSQN 2%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_issqn_3" model="account.tax.group">
            <field name="name">ISSQN 3%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_issqn_4" model="account.tax.group">
            <field name="name">ISSQN 4%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_issqn_5" model="account.tax.group">
            <field name="name">ISSQN 5%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_csll_0" model="account.tax.group">
            <field name="name">CSLL 0%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_0" model="account.tax.group">
            <field name="name">IPI 0%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_2" model="account.tax.group">
            <field name="name">IPI 2%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_3" model="account.tax.group">
            <field name="name">IPI 3%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_4" model="account.tax.group">
            <field name="name">IPI 4%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_5" model="account.tax.group">
            <field name="name">IPI 5%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_7" model="account.tax.group">
            <field name="name">IPI 7%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_8" model="account.tax.group">
            <field name="name">IPI 8%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_10" model="account.tax.group">
            <field name="name">IPI 10%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_12" model="account.tax.group">
            <field name="name">IPI 12%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_13" model="account.tax.group">
            <field name="name">IPI 13%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_15" model="account.tax.group">
            <field name="name">IPI 15%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_16" model="account.tax.group">
            <field name="name">IPI 16%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_18" model="account.tax.group">
            <field name="name">IPI 18%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_20" model="account.tax.group">
            <field name="name">IPI 20%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_22" model="account.tax.group">
            <field name="name">IPI 22%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_24" model="account.tax.group">
            <field name="name">IPI 24%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_25" model="account.tax.group">
            <field name="name">IPI 25%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_27" model="account.tax.group">
            <field name="name">IPI 27%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_30" model="account.tax.group">
            <field name="name">IPI 30%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_35" model="account.tax.group">
            <field name="name">IPI 35%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_40" model="account.tax.group">
            <field name="name">IPI 40%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_42" model="account.tax.group">
            <field name="name">IPI 42%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_45" model="account.tax.group">
            <field name="name">IPI 45%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_50" model="account.tax.group">
            <field name="name">IPI 50%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_60" model="account.tax.group">
            <field name="name">IPI 60%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_300" model="account.tax.group">
            <field name="name">IPI 300%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_330" model="account.tax.group">
            <field name="name">IPI 330%</field>
            <field name="country_id" ref="base.br"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.br"/>
    </record>

    <record id="tax_report_icmsst" model="account.tax.report.line">
        <field name="name">ICMSST</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_icmsst_1" model="account.tax.report.line">
        <field name="name">ICMSST_1</field>
        <field name="tag_name">ICMSST_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_icmsst"/>
    </record>

    <record id="tax_report_icmsst_2" model="account.tax.report.line">
        <field name="name">ICMSST_2</field>
        <field name="tag_name">ICMSST_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_icmsst"/>
    </record>

    <record id="tax_report_irpj" model="account.tax.report.line">
        <field name="name">IRPJ</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_irpj_1" model="account.tax.report.line">
        <field name="name">IRPJ_1</field>
        <field name="tag_name">IRPJ_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_irpj"/>
    </record>

    <record id="tax_report_irpj_2" model="account.tax.report.line">
        <field name="name">IRPJ_2</field>
        <field name="tag_name">IRPJ_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_irpj"/>
    </record>

    <record id="tax_report_ir" model="account.tax.report.line">
        <field name="name">IR</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_ir_1" model="account.tax.report.line">
        <field name="name">IR_1</field>
        <field name="tag_name">IR_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_ir"/>
    </record>

    <record id="tax_report_ir_2" model="account.tax.report.line">
        <field name="name">IR_2</field>
        <field name="tag_name">IR_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_ir"/>
    </record>

    <record id="tax_report_issqn" model="account.tax.report.line">
        <field name="name">ISSQN</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_issqn_1" model="account.tax.report.line">
        <field name="name">ISSQN_1</field>
        <field name="tag_name">ISSQN_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_issqn"/>
    </record>

    <record id="tax_report_issqn_2" model="account.tax.report.line">
        <field name="name">ISSQN_2</field>
        <field name="tag_name">ISSQN_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_issqn"/>
    </record>


    <record id="tax_report_csll" model="account.tax.report.line">
        <field name="name">CSLL</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_csll_1" model="account.tax.report.line">
        <field name="name">CSLL_1</field>
        <field name="tag_name">CSLL_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_csll"/>
    </record>

    <record id="tax_report_csll_2" model="account.tax.report.line">
        <field name="name">CSLL_2</field>
        <field name="tag_name">CSLL_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_csll"/>
    </record>

    <record id="tax_report_cofins" model="account.tax.report.line">
        <field name="name">COFINS</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_cofins_oper_bas" model="account.tax.report.line">
        <field name="name">Operação Tributável com Alíquota Básica</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_1" model="account.tax.report.line">
        <field name="name">COFINS_1</field>
        <field name="tag_name">COFINS_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_cofins_oper_bas"/>
    </record>

    <record id="tax_report_cofins_2" model="account.tax.report.line">
        <field name="name">COFINS_2</field>
        <field name="tag_name">COFINS_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_cofins_oper_bas"/>
    </record>

    <record id="tax_report_cofins_oper_dif" model="account.tax.report.line">
        <field name="name">Operação Tributável com Alíquota Diferenciada</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_uni" model="account.tax.report.line">
        <field name="name">Operação Tributável com Alíquota por Unidade de Medida de Produto</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_line12" model="account.tax.report.line">
        <field name="name">Operação Tributável Monofásica - Revenda a Alíquota Zero</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_sub" model="account.tax.report.line">
        <field name="name">Operação Tributável por Substituição Tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_tri" model="account.tax.report.line">
        <field name="name">Operação Tributável a Alíquota zero</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_isenta" model="account.tax.report.line">
        <field name="name">Operação Isenta da Contribuição</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_sem" model="account.tax.report.line">
        <field name="name">Operação sem Incidência da Contribuição</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_com" model="account.tax.report.line">
        <field name="name">Operação com Suspensão da Contribuição</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_outras_oper_saida" model="account.tax.report.line">
        <field name="name">Outras Operações de Saída</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_mercado" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada Exclusivamente a Receita Tributada no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="11"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_no_mercado" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada Exclusivamente a Receita Não-Tributada no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="12"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_receita" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada Exclusivamente a Receita de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="13"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_nao_tri" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="14"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_oper_export" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada a Receitas Tributadas no Mercado Interno e de Exportação </field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="15"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_oper_interno_export" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada a Receitas Não Tributadas no Mercado Interno e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="16"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_oper_no_marcado_interno_export" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="17"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_credito_oper_no_marcado" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita Tributada no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="18"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_credito_oper_tri" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita Não-Tributada no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="19"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_credito_oper_export" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="20"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_credito_oper_nao_tributadas" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="21"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_credito_oper_tributadas" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas no Mercado Interno e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="22"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_credito_nao_oper_tributadas" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada a Receitas Não-Tributadas no Mercado Interno e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="23"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_credito_oper_receitas" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="24"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_credito_outras_oper" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Outras Operações</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="25"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_oper_aqui_sem" model="account.tax.report.line">
        <field name="name">Operação de Aquisição sem Direito a Crédito</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="26"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_oper_aqui_com" model="account.tax.report.line">
        <field name="name">Operação de Aquisição com Isenção</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="27"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_oper_aqui_com_sus" model="account.tax.report.line">
        <field name="name">Operação de Aquisição com Suspensão</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="28"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_oper_aqui_zero" model="account.tax.report.line">
        <field name="name">Operação de Aquisição a Alíquota Zero</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="29"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_oper_aqui_contri" model="account.tax.report.line">
        <field name="name">Operação de Aquisição sem Incidência da Contribuição</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="30"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_oper_aqui_sub" model="account.tax.report.line">
        <field name="name">Operação de Aquisição por Substituição Tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="31"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_outras_oper_entrada" model="account.tax.report.line">
        <field name="name">Outras Operações de Entrada</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="32"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_cofins_outras_oper" model="account.tax.report.line">
        <field name="name">Outras Operações</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="33"/>
        <field name="parent_id" ref="tax_report_cofins"/>
    </record>

    <record id="tax_report_pis" model="account.tax.report.line">
        <field name="name">PIS</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_pis_oper_tri_basica" model="account.tax.report.line">
        <field name="name">Operação Tributável com Alíquota Básica</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_1" model="account.tax.report.line">
        <field name="name">PIS_1</field>
        <field name="tag_name">PIS_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_pis_oper_tri_basica"/>
    </record>

    <record id="tax_report_pis_2" model="account.tax.report.line">
        <field name="name">PIS_2</field>
        <field name="tag_name">PIS_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_pis_oper_tri_basica"/>
    </record>

    <record id="tax_report_pis_oper_tri_diferenciada" model="account.tax.report.line">
        <field name="name">Operação Tributável com Alíquota Diferenciada</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_tri_produto" model="account.tax.report.line">
        <field name="name">Operação Tributável com Alíquota por Unidade de Medida de Produto</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_tri_zero" model="account.tax.report.line">
        <field name="name">Operação Tributável Monofásica - Revenda a Alíquota Zero</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_tri_sub" model="account.tax.report.line">
        <field name="name">Operação Tributável por Substituição Tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_tri_ali_zero" model="account.tax.report.line">
        <field name="name">Operação Tributável a Alíquota Zero</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_isenta" model="account.tax.report.line">
        <field name="name">Operação Isenta da Contribuição</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_sem" model="account.tax.report.line">
        <field name="name">Operação sem Incidência da Contribuição</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_com" model="account.tax.report.line">
        <field name="name">Operação com Suspensão da Contribuição</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_outras_oper_saida" model="account.tax.report.line">
        <field name="name">Outras Operações de Saída</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_com_direito" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada Exclusivamente a Receita Tributada no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="11"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_com_credito" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada Exclusivamente a Receita Não Tributada no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="12"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_com_vinculada_ex" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada Exclusivamente a Receita de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="13"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_com_vinculada_rec" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="14"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_com_export" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada a Receitas Tributadas no Mercado Interno e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="15"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_com_nao_tri" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada a Receitas Não-Tributadas no Mercado Interno e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="16"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_com_tri_" model="account.tax.report.line">
        <field name="name">Operação com Direito a Crédito - Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno, e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="17"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_credito_presumido_tributada" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita Tributada no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="18"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_credito_presumido_nao_tributada" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita Não-Tributada no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="19"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_credito_presumido_export" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="20"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_credito_presumido_interno" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="21"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_credito_presumido_interno_export" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas no Mercado Interno e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="22"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_credito_presumido_oper_nao_tri" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada a Receitas Não-Tributadas no Mercado Interno e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="23"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_credito_presumido_oper_tri" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno, e de Exportação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="24"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_credito_presumido_outras_oper" model="account.tax.report.line">
        <field name="name">Crédito Presumido - Outras Operações</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="25"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_aqui_sem" model="account.tax.report.line">
        <field name="name">Operação de Aquisição sem Direito a Crédito</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="26"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_com_ise" model="account.tax.report.line">
        <field name="name">Operação de Aquisição com Isenção</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="27"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_aqui_com" model="account.tax.report.line">
        <field name="name">Operação de Aquisição com Suspensão</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="28"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_aqui_zero" model="account.tax.report.line">
        <field name="name">Operação de Aquisição a Alíquota Zero</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="29"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_aqui_inc" model="account.tax.report.line">
        <field name="name">Operação de Aquisição sem Incidência da Contribuição</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="30"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_oper_aqui_sub" model="account.tax.report.line">
        <field name="name">Operação de Aquisição por Substituição Tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="31"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_outras_oper_entrada" model="account.tax.report.line">
        <field name="name">Outras Operações de Entrada</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="32"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_pis_outras_oper" model="account.tax.report.line">
        <field name="name">Outras Operações</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="33"/>
        <field name="parent_id" ref="tax_report_pis"/>
    </record>

    <record id="tax_report_ipi" model="account.tax.report.line">
        <field name="name">IPI</field>
        <field name="code">BRTAX07</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
    </record>

    <record id="tax_report_ipi_extrada_com" model="account.tax.report.line">
        <field name="name">Entrada com recuperação de crédito</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_1" model="account.tax.report.line">
        <field name="name">IPI_1</field>
        <field name="tag_name">IPI_1</field>
        <field name="code">BRTAX07_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_ipi_extrada_com"/>
    </record>

    <record id="tax_report_ipi_extrada_tributada" model="account.tax.report.line">
        <field name="name">Entrada tributada com alíquota zero</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_2" model="account.tax.report.line">
        <field name="name">IPI_2</field>
        <field name="tag_name">IPI_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_ipi_extrada_tributada"/>
    </record>

    <record id="tax_report_ipi_entrada_isenta" model="account.tax.report.line">
        <field name="name">Entrada isenta</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_entrada_nao_tributada" model="account.tax.report.line">
        <field name="name">EEntrada não-tributada</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_entrada_imune" model="account.tax.report.line">
        <field name="name">Entrada imune</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_entrada_com_suspensao" model="account.tax.report.line">
        <field name="name">Entrada com suspensão</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_outras_entradas" model="account.tax.report.line">
        <field name="name">Outras entradas</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_saida_tributada" model="account.tax.report.line">
        <field name="name">Saída tributada</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_saida_tributada_com" model="account.tax.report.line">
        <field name="name">Saída tributada com alíquota zero</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_saida_isenta" model="account.tax.report.line">
        <field name="name">Saída isenta</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_saida_nao_tributada" model="account.tax.report.line">
        <field name="name">Saída não-tributada</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="11"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_saida_imune" model="account.tax.report.line">
        <field name="name">Saída imune</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="12"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_saida_com" model="account.tax.report.line">
        <field name="name">Saída com suspensão</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="13"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_ipi_outras_saidas" model="account.tax.report.line">
        <field name="name">Outras saídas</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="14"/>
        <field name="parent_id" ref="tax_report_ipi"/>
    </record>

    <record id="tax_report_icms" model="account.tax.report.line">
        <field name="name">ICMS</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
    </record>

    <record id="tax_report_icms_tributada" model="account.tax.report.line">
        <field name="name">Tributada integralmente</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_1" model="account.tax.report.line">
        <field name="name">ICMS_1</field>
        <field name="tag_name">ICMS_1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_icms_tributada"/>
    </record>

    <record id="tax_report_icms_tributada_com" model="account.tax.report.line">
        <field name="name">Tributada e com cobrança do ICMS por substituição tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_2" model="account.tax.report.line">
        <field name="name">ICMS_2</field>
        <field name="tag_name">ICMS_2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_icms_tributada_com"/>
    </record>

    <record id="tax_report_icms_com_red" model="account.tax.report.line">
        <field name="name">Com redução de base de cálculo</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_isenta_tributada" model="account.tax.report.line">
        <field name="name">Isenta ou não tributada e com cobrança do ICMS por substituição tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_isenta" model="account.tax.report.line">
        <field name="name">Isenta</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

     <record id="tax_report_icms_nao_tributada" model="account.tax.report.line">
        <field name="name">Não tributada</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_suspensao" model="account.tax.report.line">
        <field name="name">Suspensão</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

     <record id="tax_report_icms_diferimento" model="account.tax.report.line">
        <field name="name">Diferimento</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_cobrado" model="account.tax.report.line">
        <field name="name">ICMS cobrado anteriormente por substituição tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_reducao" model="account.tax.report.line">
        <field name="name">Com redução de base de cálculo e cobrança do ICMS por substituição tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_outras" model="account.tax.report.line">
        <field name="name">Outras</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="11"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_com" model="account.tax.report.line">
        <field name="name">Simples Nacional - Tributada pelo Simples Nacional com permissão de crédito</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="12"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_sem" model="account.tax.report.line">
        <field name="name">Simples Nacional - Tributada pelo Simples Nacional sem permissão de crédito</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="13"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_isencao" model="account.tax.report.line">
        <field name="name">Simples Nacional - Isenção do ICMS no Simples Nacional para faixa de receita bruta</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="14"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_tributada" model="account.tax.report.line">
        <field name="name">Simples Nacional - Tributada pelo Simples Nacional com permissão de crédito e com cobrança do ICMS por substituição tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="15"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_tributada_pelo" model="account.tax.report.line">
        <field name="name">Simples Nacional - Tributada pelo Simples Nacional sem permissão de crédito e com cobrança do ICMS por substituição tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="16"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_isencao_icms" model="account.tax.report.line">
        <field name="name">Simples Nacional - Isenção do ICMS no Simples Nacional para faixa de receita bruta e com cobrança do ICMS por substituição tributária</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="17"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_imune" model="account.tax.report.line">
        <field name="name">Simples Nacional - Imune</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="18"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_nao_tributada" model="account.tax.report.line">
        <field name="name">Simples Nacional - Não tributada pelo Simples Nacional</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="19"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_icms" model="account.tax.report.line">
        <field name="name">Simples Nacional - ICMS cobrado anteriormente por substituição tributária (substituído) ou por antecipação</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="20"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>

    <record id="tax_report_icms_simples_nacional_outras" model="account.tax.report.line">
        <field name="name">Simples Nacional - Outros</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="21"/>
        <field name="parent_id" ref="tax_report_icms"/>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="ch_br_3_01_03_01_01_03_00" model="account.account.template">
            <field name="code">3.01.03.01.01.03.00</field>
            <field name="name">ganho cambial</field>
            <field name="user_type_id" ref="account.data_account_type_other_income"/>
            <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
         </record>

         <record id="ch_br_3_01_03_01_03_03_00" model="account.account.template">
            <field name="code">3.01.03.01.03.03.00</field>
            <field name="name">Perda cambial</field>
            <field name="user_type_id" ref="account.data_account_type_expenses"/>
            <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
         </record>

		<record id="l10n_br_account_chart_template" model="account.chart.template">
			<field name="property_account_receivable_id" ref="account_template_101050200"/>
			<field name="property_account_payable_id" ref="account_template_201010100" />
			<field name="property_account_expense_categ_id" ref="account_template_3010103010000" />
			<field name="property_account_income_categ_id" ref="account_template_3010101010200" />
            <field name="income_currency_exchange_account_id" ref="ch_br_3_01_03_01_01_03_00"/>
            <field name="expense_currency_exchange_account_id" ref="ch_br_3_01_03_01_03_03_00"/>
			<field name="default_pos_receivable_account_id" ref="account_template_101050201" />
		</record>

		<record id="tax_template_out_ipi" model="account.tax.template">
			<field name="description">IPI </field>
			<field name="name">IPI Saída</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi2" model="account.tax.template">
			<field name="description">IPI 2%</field>
			<field name="name">IPI Saída 2%</field>
			<field name="amount">2</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_2"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi3" model="account.tax.template">
			<field name="description">IPI 3%</field>
			<field name="name">IPI Saída 3%</field>
			<field name="amount">3</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_3"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi4" model="account.tax.template">
			<field name="description">IPI 4%</field>
			<field name="name">IPI Saída 4%</field>
			<field name="amount">4</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_4"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi5" model="account.tax.template">
			<field name="description">IPI 5%</field>
			<field name="name">IPI Saída 5%</field>
			<field name="amount">5</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_5"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi7" model="account.tax.template">
			<field name="description">IPI 7%</field>
			<field name="name">IPI Saída 7%</field>
			<field name="amount">7</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_7"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
        </record>

		<record id="tax_template_out_ipi8" model="account.tax.template">
			<field name="description">IPI 8%</field>
			<field name="name">IPI Saída 8%</field>
			<field name="amount">8</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_8"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi10" model="account.tax.template">
			<field name="description">IPI 10%</field>
			<field name="name">IPI Saída 10%</field>
			<field name="amount">10</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_10"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi12" model="account.tax.template">
			<field name="description">IPI 12%</field>
			<field name="name">IPI Saída 12%</field>
			<field name="amount">12</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_12"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi13" model="account.tax.template">
			<field name="description">IPI 13%</field>
			<field name="name">IPI Saída 13%</field>
			<field name="amount">13</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_13"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi15" model="account.tax.template">
			<field name="description">IPI 15%</field>
			<field name="name">IPI Saída 15%</field>
			<field name="amount">15</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_15"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi16" model="account.tax.template">
			<field name="description">IPI 16%</field>
			<field name="name">IPI Saída 16%</field>
			<field name="amount">16</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_16"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi18" model="account.tax.template">
			<field name="description">IPI 18%</field>
			<field name="name">IPI Saída 18%</field>
			<field name="amount">18</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_18"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi20" model="account.tax.template">
			<field name="description">IPI 20%</field>
			<field name="name">IPI Saída 20%</field>
			<field name="amount">20</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_20"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
		</record>

		<record id="tax_template_out_ipi22" model="account.tax.template">
			<field name="description">IPI 22%</field>
			<field name="name">IPI Saída 22%</field>
			<field name="amount">22</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_22"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi24" model="account.tax.template">
			<field name="description">IPI 24%</field>
			<field name="name">IPI Saída 24%</field>
			<field name="amount">24</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_24"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi25" model="account.tax.template">
			<field name="description">IPI 25%</field>
			<field name="name">IPI Saída 25%</field>
			<field name="amount">25</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_25"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi27" model="account.tax.template">
			<field name="description">IPI 27%</field>
			<field name="name">IPI Saída 27%</field>
			<field name="amount">27</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_27"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi30" model="account.tax.template">
			<field name="description">IPI 30%</field>
			<field name="name">IPI Saída 30%</field>
			<field name="amount">30</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_30"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi35" model="account.tax.template">
			<field name="description">IPI 35%</field>
			<field name="name">IPI Saída 35%</field>
			<field name="amount">35</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_35"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
		</record>

		<record id="tax_template_out_ipi40" model="account.tax.template">
			<field name="description">IPI 40%</field>
			<field name="name">IPI Saída 40%</field>
			<field name="amount">40</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_40"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi42" model="account.tax.template">
			<field name="description">IPI 42%</field>
			<field name="name">IPI Saída 42%</field>
			<field name="amount">42</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_42"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi45" model="account.tax.template">
			<field name="description">IPI 45%</field>
			<field name="name">IPI Saída 45%</field>
			<field name="amount">45</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_45"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi50" model="account.tax.template">
			<field name="description">IPI 50%</field>
			<field name="name">IPI Saída 50%</field>
			<field name="amount">50</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_50"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi60" model="account.tax.template">
			<field name="description">IPI 60%</field>
			<field name="name">IPI Saída 60%</field>
			<field name="amount">60</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_60"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi300" model="account.tax.template">
			<field name="description">IPI 300%</field>
			<field name="name">IPI Saída 300%</field>
			<field name="amount">300</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_300"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_ipi330" model="account.tax.template">
			<field name="description">IPI 330%</field>
			<field name="name">IPI Saída 330%</field>
			<field name="amount">330</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_330"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

       	<record id="tax_template_in_ipi" model="account.tax.template">
			<field name="description">IPI</field>
			<field name="name">IPI Entrada</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi2" model="account.tax.template">
			<field name="description">IPI 2%</field>
			<field name="name">IPI Entrada 2%</field>
			<field name="amount">2</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_2"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi3" model="account.tax.template">
			<field name="description">IPI 3%</field>
			<field name="name">IPI Entrada 3%</field>
			<field name="amount">3</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_3"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi4" model="account.tax.template">
			<field name="description">IPI 4%</field>
			<field name="name">IPI Entrada 4%</field>
			<field name="amount">4</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_4"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi5" model="account.tax.template">
			<field name="description">IPI 5%</field>
			<field name="name">IPI Entrada 5%</field>
			<field name="amount">5</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_5"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi7" model="account.tax.template">
			<field name="description">IPI 7%</field>
			<field name="name">IPI Entrada 7%</field>
			<field name="amount">7</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_7"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
        	</record>

		<record id="tax_template_in_ipi8" model="account.tax.template">
			<field name="description">IPI 8%</field>
			<field name="name">IPI Entrada 8%</field>
			<field name="amount">8</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_8"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi10" model="account.tax.template">
			<field name="description">IPI 10%</field>
			<field name="name">IPI Entrada 10%</field>
			<field name="amount">10</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_10"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi12" model="account.tax.template">
			<field name="description">IPI 12%</field>
			<field name="name">IPI Entrada 12%</field>
			<field name="amount">12</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_12"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi13" model="account.tax.template">
			<field name="description">IPI 13%</field>
			<field name="name">IPI Entrada 13%</field>
			<field name="amount">13</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_13"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi15" model="account.tax.template">
			<field name="description">IPI 15%</field>
			<field name="name">IPI Entrada 15%</field>
			<field name="amount">15</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_15"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi16" model="account.tax.template">
			<field name="description">IPI 16%</field>
			<field name="name">IPI Entrada 16%</field>
			<field name="amount">16</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_16"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi18" model="account.tax.template">
			<field name="description">IPI 18%</field>
			<field name="name">IPI Entrada 18%</field>
			<field name="amount">18</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_18"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi20" model="account.tax.template">
			<field name="description">IPI 20%</field>
			<field name="name">IPI Entrada 20%</field>
			<field name="amount">20</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_20"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
        </record>

		<record id="tax_template_in_ipi22" model="account.tax.template">
			<field name="description">IPI 22%</field>
			<field name="name">IPI Entrada 22%</field>
			<field name="amount">22</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_22"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi24" model="account.tax.template">
			<field name="description">IPI 24%</field>
			<field name="name">IPI Entrada 24%</field>
			<field name="amount">24</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_24"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi25" model="account.tax.template">
			<field name="description">IPI 25%</field>
			<field name="name">IPI Entrada 25%</field>
			<field name="amount">25</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_25"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi27" model="account.tax.template">
			<field name="description">IPI 27%</field>
			<field name="name">IPI Entrada 27%</field>
			<field name="amount">27</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_27"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi30" model="account.tax.template">
			<field name="description">IPI 30%</field>
			<field name="name">IPI Entrada 30%</field>
			<field name="amount">30</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_30"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi35" model="account.tax.template">
			<field name="description">IPI 35%</field>
			<field name="name">IPI Entrada 35%</field>
			<field name="amount">35</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_35"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
		</record>

		<record id="tax_template_in_ipi40" model="account.tax.template">
			<field name="description">IPI 40%</field>
			<field name="name">IPI Entrada 40%</field>
			<field name="amount">40</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_40"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi42" model="account.tax.template">
			<field name="description">IPI 42%</field>
			<field name="name">IPI Entrada 42%</field>
			<field name="amount">42</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_42"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi45" model="account.tax.template">
			<field name="description">IPI 45%</field>
			<field name="name">IPI Entrada 45%</field>
			<field name="amount">45</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_45"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi50" model="account.tax.template">
			<field name="description">IPI 50%</field>
			<field name="name">IPI Entrada 50%</field>
			<field name="amount">50</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_50"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi60" model="account.tax.template">
			<field name="description">IPI 60%</field>
			<field name="name">IPI Entrada 60%</field>
			<field name="amount">60</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_60"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi300" model="account.tax.template">
			<field name="description">IPI 300%</field>
			<field name="name">IPI Entrada 300%</field>
			<field name="amount">300</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_300"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_ipi330" model="account.tax.template">
			<field name="description">IPI 330%</field>
			<field name="name">IPI Entrada 330%</field>
			<field name="amount">330</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ipi_330"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050502'),
	                'minus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ipi_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010301'),
	                'plus_report_line_ids': [ref('tax_report_ipi_2')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_icms_interno" model="account.tax.template">
			<field name="description">ICMS Interno</field>
			<field name="name">ICMS Saida Interno</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_icms_externo" model="account.tax.template">
			<field name="description">ICMS Externo</field>
			<field name="name">ICMS Saida Externo</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_icms_subist" model="account.tax.template">
			<field name="description">ICMS Subist</field>
			<field name="name">ICMS Saida Subist</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icmsst_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icmsst_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_icms_externo7" model="account.tax.template">
			<field name="description">ICMS Externo 7%</field>
			<field name="name">ICMS Saida Externo 7%</field>
			<field name="amount">7</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_7"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010302'),
	                'plus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050505'),
	                'minus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_icms_externo12" model="account.tax.template">
			<field name="description">ICMS Externo 12%</field>
			<field name="name">ICMS Saida Externo 12%</field>
			<field name="amount">12</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_12"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010302'),
	                'plus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050505'),
	                'minus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_icms_interno19" model="account.tax.template">
			<field name="description">ICMS Interno 19%</field>
			<field name="name">ICMS Saida Interno 19%</field>
			<field name="amount">19</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_19"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010302'),
	                'plus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050505'),
	                'minus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_icms_interno26" model="account.tax.template">
			<field name="description">ICMS Interno 26%</field>
			<field name="name">ICMS Saida Interno 26%</field>
			<field name="amount">26</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_26"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010302'),
	                'plus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050505'),
	                'minus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
       	</record>

       	<record id="tax_template_in_icms_interno" model="account.tax.template">
			<field name="description">ICMS Interno</field>
			<field name="name">ICMS Entrada Interno</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_icms_externo" model="account.tax.template">
			<field name="description">ICMS Externo</field>
			<field name="name">ICMS Entrada Externo</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_icms_subist" model="account.tax.template">
			<field name="description">ICMS Subist</field>
			<field name="name">ICMS Entrada Subist</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="0" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icmsst_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icmsst_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_icms_externo7" model="account.tax.template">
			<field name="description">ICMS Externo 7%</field>
			<field name="name">ICMS Entrada Externo 7%</field>
			<field name="amount">7</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_7"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050505'),
	                'minus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010302'),
	                'plus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_icms_externo12" model="account.tax.template">
			<field name="description">ICMS Externo 12%</field>
			<field name="name">ICMS Entrada Externo 12%</field>
			<field name="amount">12</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_12"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050505'),
	                'minus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010302'),
	                'plus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_icms_interno19" model="account.tax.template">
			<field name="description">ICMS Interno 19%</field>
			<field name="name">ICMS Entrada Interno 19%</field>
			<field name="amount">19</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_19"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050505'),
	                'minus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010302'),
	                'plus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_icms_interno26" model="account.tax.template">
			<field name="description">ICMS Interno 26%</field>
			<field name="name">ICMS Entrada Interno 26%</field>
			<field name="amount">26</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_icms_26"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050505'),
	                'minus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_icms_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010302'),
	                'plus_report_line_ids': [ref('tax_report_icms_1')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_pis" model="account.tax.template">
			<field name="description">PIS</field>
			<field name="name">PIS Saida</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_pis_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_pis_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_pis_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_pis065" model="account.tax.template">
			<field name="description">PIS 0,65%</field>
			<field name="name">PIS Saida 0,65%</field>
			<field name="amount">0.65</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_pis_065"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_pis_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010500'),
	                'plus_report_line_ids': [ref('tax_report_pis_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_pis_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050503'),
	                'minus_report_line_ids': [ref('tax_report_pis_1')],
	            }),
	        ]"/>
       	</record>

       	<record id="tax_template_in_pis" model="account.tax.template">
			<field name="description">PIS</field>
			<field name="name">PIS Entrada</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_pis_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_pis_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_pis_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_pis065" model="account.tax.template">
			<field name="description">PIS 0,65%</field>
			<field name="name">PIS Entrada 0,65%</field>
			<field name="amount">0.65</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_pis_065"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_pis_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050503'),
	                'minus_report_line_ids': [ref('tax_report_pis_1')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_pis_2')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010500'),
	                'plus_report_line_ids': [ref('tax_report_pis_1')],
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_cofins" model="account.tax.template">
			<field name="description">COFINS</field>
			<field name="name">COFINS Saida</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_cofins_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_cofins_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_cofins_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_out_cofins3" model="account.tax.template">
			<field name="description">COFINS 3%</field>
			<field name="name">COFINS Saida 3%</field>
			<field name="amount">3</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_cofins_3"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_cofins_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010500'),
	                'plus_report_line_ids': [ref('tax_report_cofins_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_cofins_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050503'),
	                'minus_report_line_ids': [ref('tax_report_cofins_2')],
	            }),
	        ]"/>
        </record>

        <record id="tax_template_in_cofins" model="account.tax.template">
			<field name="description">COFINS</field>
			<field name="name">COFINS Entrada</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_cofins_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_cofins_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_cofins_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
       	</record>

		<record id="tax_template_in_cofins3" model="account.tax.template">
			<field name="description">COFINS 3%</field>
			<field name="name">COFINS Entrada 3%</field>
			<field name="amount">3</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_cofins_3"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_cofins_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_101050503'),
	                'minus_report_line_ids': [ref('tax_report_cofins_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_cofins_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'account_id': ref('account_template_201010500'),
	                'plus_report_line_ids': [ref('tax_report_cofins_2')],
	            }),
	        ]"/>
        </record>

		<record id="tax_template_out_irpj" model="account.tax.template">
			<field name="description">IRPJ</field>
			<field name="name">IRPJ</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_irpj_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_irpj_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_irpj_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
        </record>

		<record id="tax_template_out_ir" model="account.tax.template">
			<field name="description">IR</field>
			<field name="name">IR</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_ir_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_ir_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_ir_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
        </record>

		<record id="tax_template_out_issqn" model="account.tax.template">
			<field name="description">ISSQN</field>
			<field name="name">ISSQN Saida</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_issqn_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
        </record>

        <record id="tax_template_out_issqn1" model="account.tax.template">
			<field name="description">ISSQN 1%</field>
			<field name="name">ISSQN Saida 1%</field>
			<field name="amount">1</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_issqn_1"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'plus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'minus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
        </record>

        <record id="tax_template_out_issqn2" model="account.tax.template">
			<field name="description">ISSQN 2%</field>
			<field name="name">ISSQN Saida 2%</field>
			<field name="amount">2</field>
            <field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_issqn_2"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'plus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'minus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
        </record>
		<record id="tax_template_in_issqn2" model="account.tax.template">
			<field name="description">ISSQN 2%</field>
			<field name="name">ISSQN Entrada 2%</field>
			<field name="amount">2</field>
			<field name="type_tax_use">purchase</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_issqn_2"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'minus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'plus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
		</record>

        <record id="tax_template_out_issqn3" model="account.tax.template">
			<field name="description">ISSQN 3%</field>
			<field name="name">ISSQN Saida 3%</field>
			<field name="amount">3</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_issqn_3"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'plus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'minus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
        </record>

        <record id="tax_template_out_issqn4" model="account.tax.template">
			<field name="description">ISS 4%</field>
			<field name="name">ISSQN Saida 4%</field>
			<field name="amount">4</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_issqn_4"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'plus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'minus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
        </record>

        <record id="tax_template_out_issqn5" model="account.tax.template">
			<field name="description">ISSQN 5%</field>
			<field name="name">ISSQN Saida 5%</field>
			<field name="amount">5</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_issqn_5"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'plus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_issqn_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	                'minus_report_line_ids': [ref('tax_report_issqn_2')],
	            }),
	        ]"/>
        </record>

		<record id="tax_template_out_csll" model="account.tax.template">
			<field name="description">CSLL</field>
			<field name="name">CSLL</field>
			<field name="amount">0.00</field>
			<field name="type_tax_use">sale</field>
			<field eval="0" name="price_include"/>
			<field eval="1" name="tax_discount"/>
			<field ref="l10n_br_account_chart_template" name="chart_template_id"/>
			<field name="tax_group_id" ref="tax_group_csll_0"/>
			<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'plus_report_line_ids': [ref('tax_report_csll_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
	        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'base',
	                'minus_report_line_ids': [ref('tax_report_csll_1')],
	            }),
	            (0,0, {
	                'factor_percent': 100,
	                'repartition_type': 'tax',
	            }),
	        ]"/>
        </record>

        <data noupdate="1">
            <function model="account.chart.template" name="try_loading">
                <value eval="[ref('l10n_br.l10n_br_account_chart_template')]"/>
            </function>
        </data>
</odoo>

```

## File: data\l10n_br_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_br_statements_menu" name="Brazil" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

	<record id="l10n_br_account_chart_template" model="account.chart.template">
		<field name="name">Planilha de Contas Brasileira</field>
		<field name="code_digits">6</field>
        <field name="bank_account_code_prefix">1.01.01.02.00</field>
        <field name="cash_account_code_prefix">1.01.01.01.00</field>
        <field name="transfer_account_code_prefix">1.01.01.12.00</field>
		<field name="currency_id" ref="base.BRL"/>
        <field name="country_id" ref="base.br"/>
	</record>
</odoo>

```

## File: models\account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountTaxTemplate(models.Model):
    """ Add fields used to define some brazilian taxes """
    _inherit = 'account.tax.template'

    tax_discount = fields.Boolean(string='Discount this Tax in Prince',
                                    help="Mark it for (ICMS, PIS e etc.).")
    base_reduction = fields.Float(string='Redution', digits=0, required=True,
                                    help="Um percentual decimal em % entre 0-1.", default=0)
    amount_mva = fields.Float(string='MVA Percent', digits=0, required=True,
                                help="Um percentual decimal em % entre 0-1.", default=0)


class AccountTax(models.Model):
    """ Add fields used to define some brazilian taxes """
    _inherit = 'account.tax'

    tax_discount = fields.Boolean(string='Discount this Tax in Prince', 
                                  help="Mark it for (ICMS, PIS e etc.).")
    base_reduction = fields.Float(string='Redution', digits=0, required=True,
                                  help="Um percentual decimal em % entre 0-1.", default=0)
    amount_mva = fields.Float(string='MVA Percent', digits=0, required=True,
                              help="Um percentual decimal em % entre 0-1.", default=0)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2009  Renato Lima - Akretion

from . import account

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="5.9" y="4.69" width="49.2" height="37.31" maskUnits="userSpaceOnUse">
      <rect x="6.19" y="7.57" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
    </mask>
    <symbol id="c" data-name="account icon" viewBox="0 0 106 106">
      <g style="mask: url(#a)">
        <g>
          <path d="M0,0H106V106H0Z" style="fill: #5a5a64;fill-rule: evenodd"/>
          <path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill: #fff;fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <g>
            <path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill: #393939;fill-rule: evenodd;opacity: 0.324000000953674;isolation: isolate"/>
            <g style="opacity: 0.30000000000000004">
              <g>
                <path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/>
                <path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/>
                <path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/>
                <path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/>
                <path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/>
                <path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/>
                <path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/>
              </g>
              <path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/>
            </g>
            <path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill: #a8a9ab"/>
            <g>
              <path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill: #a8a9ab"/>
              <path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill: #a8a9ab"/>
              <path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill: #a8a9ab"/>
              <path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill: #a8a9ab"/>
              <path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill: #a8a9ab"/>
              <path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill: #a8a9ab"/>
              <path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill: #a8a9ab"/>
            </g>
          </g>
        </g>
      </g>
    </symbol>
  </defs>
  <g>
    <use width="106" height="106" transform="translate(-0.07 0)" xlink:href="#c"/>
    <rect x="6.2" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="800" height="560" transform="translate(5.9 4.69) scale(0.06 0.07)" xlink:href="data:image/png;base64,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+TSzR+gPVsASWGFC7ZyVI4EbZumlb5yIiJzRFEJERLYpj1zHgZTAIneDAAspGlqDoQXD2WOs0OMXv3IlN9x6HV9+bicxBuslSs/xObjlybO48uBefv2rl1DCGOyYp8/0+0KethAmcE+ElbVWwiIisj0phIiIbFOJVLdWmQNOwUnFKBRSTIBMwRg3K6TZI9z69Bu45uYb+eQD74aUSQS/+OWL+fD+6/jKodfRzB1l3EywcFqMsETGsW4LV0Suc02yrU1FFxGR7al5uQuIiMiZKRN4gajjB7GogwMtoNDUrVYFwiYk69GbOcLyZJZfv/8SDhzaxXLAl37wFmK8xEzvMJPSp7UJTekRXqvWpyMGa+RYP++llRARke1NIURERE4qOzh9zCCXHjPNhMnsCgefPo8wpz93DAcmbQ8a8NJbm8qupXYRETkZhRARETkhJ8g4hcAKJHMmJeMOg/EK1hoWMKHgyfG2rm4Ug+SZKCo+FxGRE9OJKhEROaGIwAPwQkSLR2DJKRjR1XVkMp4KFoU2WkgOkSihmg8RETk5rYSIiMhJJCCgOO6JHIVSgmROSYUWh2igLWDgTd2KhbXgCbLqPkRE5MS0EiIiIiIiIptKIURERE4i11a6NsHJ9UOescgM2kTTBlZarOkuV4KIDN5A0SqIiIicnEKIiIicmCesgEePHLXRrtMnm7NqLTkZ4FjpUUrX5temAaS81HcWEZFtTjUhIiJyQhFBdoCCW+10VaJgZhgJy93no+CeKBSIafjo6klEREROQCFEREROyNYyhBNdoDCAqP8KW79MRGFjPyzTMEIREXkJ2o4lIiIiIiKbSiFEREREREQ2lUKIiIiIiIhsKoUQERERERHZVAohIiIiIiKyqRRCRERERERkUymEiIiIiIjIplIIERERERGRTaUQIiIiIiIim0ohRERERERENpVCiIiIiIiIbCqFEBERERER2VQKISIiIiIisqkUQkREREREZFMphIiIiIiIyKZSCBERERERkU2lECIiIiIiIptKIURERERERDaVQoiIiIiIiGwqhRAREREREdlUCiEiIiIiIrKpFEJERERERGRTKYSIiIiIiMimUggREREREZFNpRAiIiIiIiKbSiFEREREREQ2lUKIiIiIiIhsKoUQERERERHZVAohIiIiIiKyqRRCRERERERkUymEiIiIiIjIplIIERERERGRTaUQIiIiIiIim0ohRERERERENpVCiIiIiIiIbCqFEBERERER2VQKISIiIiIisqkUQkREREREZFMphIiIiIiIyKZSCBERERERkU2lECIiIiIiIptKIURERERERDaVQ+AYABFB2PonN74vIiIiIiLycqYZwswIqxkD6DJHfb8BKAQ+vVApGIZZF0y6C4qIiIiIiLyctRxRgiAwdyxq5phqACy6sBHg3YWi1AtpMURERERERH5k3cqHmeFmUOqyxsZc4REB3qWVDcslYax9XERERERE5EfiduISD7e1rNFYl06g7tOqW7K67Vkl1pZTREREREREXk6U6Ko/AsxqxiCgBNbUbKHuWCIiIiIisqmagtVtVyVqsUis79cyW18yEREREREReTkbM0TEhnoQN0qXNJrl5WUigblB1O1Xxob6EO3GEhERERGRH1HtiGXrza+gqweB5eVlAJpz53ayMFkhIlgr/yixoShdKURERERERH5UXfA4QaYYj/oA/H9WtGbZNWHLpwAAAABJRU5ErkJggg=="/>
    </g>
  </g>
</svg>

```

## File: views\account_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
		<!-- Include fields created in account.tax and account.tax.template form views -->
		<record model="ir.ui.view" id="view_l10n_br_account_tax_template_form">
			<field name="name">l10n_br_account.tax.template.form</field>
			<field name="model">account.tax.template</field>
			<field name="inherit_id" ref="account.view_account_tax_template_form"/>
			<field name="arch" type="xml">
				<field position="after" name="price_include">
					<field name="tax_discount"/>
			    </field>
				<field position="after" name="tax_discount">
					<field name="base_reduction" widget="monetary"/>
					<field name="amount_mva" widget="monetary"/>
			    </field>
			</field>
		</record>

		<record model="ir.ui.view" id="view_l10n_br_account_tax_form">
			<field name="name">l10n_br_account.tax.form</field>
			<field name="model">account.tax</field>
			<field name="inherit_id" ref="account.view_tax_form"/>
			<field name="arch" type="xml">
				<field position="after" name="price_include">
					<field name="tax_discount" attrs="{'invisible': [('country_code', '!=', 'BR')]}"/>
			    </field>
			    <field position="after" name="tax_discount">
					<field name="base_reduction" widget="monetary" attrs="{'invisible': [('country_code', '!=', 'BR')]}"/>
					<field name="amount_mva" widget="monetary" attrs="{'invisible': [('country_code', '!=', 'BR')]}"/>
			    </field>
			</field>
		</record>
</odoo>

```

