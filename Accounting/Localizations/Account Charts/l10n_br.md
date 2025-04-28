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
    'depends': ['account', 'base_vat'],
    'data': [
        'data/l10n_br_chart_data.xml',
        'data/account.account.template.csv',
        'data/account_chart_template_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/fiscal_templates_data.xml',
        'data/account_fiscal_position_tax_template_data.xml',
        'views/account_view.xml',
        'views/account_fiscal_position_views.xml',
        'views/res_company_views.xml',
        'views/res_partner_views.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,note,account_type,reconcile,chart_template_id:id
account_template_101010101,1.01.01.01.01,"Caixa Matriz","Contas que registram valores em dinheiro e em cheques em caixa, recebidos e ainda não depositados, pagáveis irrestrita e imediatamente do estabelecimento matriz.",asset_cash,,l10n_br_account_chart_template
account_template_101010102,1.01.01.01.02,"Caixa Filiais","Contas que registram valores em dinheiro e em cheques em caixa, recebidos e ainda não depositados, pagáveis irrestrita e imediatamente dos estabelecimentos filiais.",asset_cash,,l10n_br_account_chart_template
account_template_101010201,1.01.01.02.01,"Bancos Conta Movimento - No País","Contas que registram disponibilidades, mantidas em instituições financeiras no país, não classificáveis em outras contas deste plano referencial. Aplicações financeiras devem receber classificação entre as contas específicas de instrumentos financeiros, p.ex. grupo 1.01.01.05.",asset_current,,l10n_br_account_chart_template
account_template_101010202,1.01.01.02.02,"Bancos Conta Movimento - No Exterior","Contas que registram disponibilidades, mantidas em instituições financeiras no exterior, não classificáveis em outras contas deste plano referencial. Aplicações financeiras devem receber classificação entre as contas específicas de instrumentos financeiros, p.ex. grupo 1.01.01.09.",asset_current,,l10n_br_account_chart_template
account_template_101010401,1.01.01.04.01,"Numerários em Trânsito","Contas que registram valores de numerários em trânsito decorrentes de remessas e/ou recebimentos para filiais, depósitos ou semelhantes, por meio de cheques, ordem de pagamentos etc., ou, ainda, de clientes ou terceiros.",asset_receivable,TRUE,l10n_br_account_chart_template
account_template_101010402,1.01.01.04.02,"Numerários em Trânsito (PoS)","PoS. Contas que registram valores de numerários em trânsito decorrentes de remessas e/ou recebimentos para filiais, depósitos ou semelhantes, por meio de cheques, ordem de pagamentos etc., ou, ainda, de clientes ou terceiros.",asset_receivable,TRUE,l10n_br_account_chart_template
account_template_101010501,1.01.01.05.01,"Títulos para Negociação - Mensurados a Valor Justo Por Meio do Resultado (VJPR) - No País","Contas que registram ativos financeiros, cuja contraparte ou ambiente negocial tenham sede ou domicílio no país, sobre os quais há a intenção de negociação no curto prazo ou se a mensuração pelo valor justo diminuir ou eliminar alguma inconsistência de mensuração de acordo com a gestão financeira da empresa (fair value option) e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Os derivativos utilizados como hedge devem ser registrados no grupo 1.01.01.06.",asset_current,,l10n_br_account_chart_template
account_template_101010502,1.01.01.05.02,"Títulos Disponíveis para Venda - No País","Contas que registram ativos financeiros, cuja contraparte ou ambiente negocial tenham sede ou domicílio no país, sobre os quais não há definição de quando nem quais condições vai negociá-los e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Contrapartida das alterações no seu valor justo, bem como custos de transação, devem ser reconhecidos no Patrimônio Líquido até a realização do ativo.",asset_current,,l10n_br_account_chart_template
account_template_101010503,1.01.01.05.03,"Títulos Mantidos até o Vencimento - No País","Contas que registram ativos financeiros, cuja contraparte ou ambiente negocial tenham sede ou domicílio no país, não derivativos, com pagamentos fixos ou determináveis e com vencimento fixo, para os quais há a intenção e a capacidade de se manter até o vencimento e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Esses títulos são avaliados pelo método de custo amortizado, com os custos de transação capitalizados ao valor do ativo. Ações e outros títulos patrimonias não devem receber esta classificação. Títulos públicos e de renda fixa constumam receber esta classificação.",asset_current,,l10n_br_account_chart_template
account_template_101010510,1.01.01.05.10,"Debêntures emitidas por Partes Relacionadas - No País","Contas que registram as debêntures emitidas por empresas com sede no país, relacionadas com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12. Independente da base de mensuração utilizada ou se tais Valores Mobiliários poderiam ser classificáveis em outras contas mais genéricas de ativos financeiros.",asset_current,,l10n_br_account_chart_template
account_template_101010511,1.01.01.05.11,"Debêntures Emitidas por Partes Não Relacionadas - No País","Contas que registram as debêntures emitidas por empresas com sede no país, não relacionadas com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, independente da base de mensuração utilizada ou se tais Valores Mobiliários poderiam ser classificáveis em outras contas mais genéricas de ativos financeiros.",asset_current,,l10n_br_account_chart_template
account_template_101010515,1.01.01.05.15,"Outros Empréstimos e Recebíveis - No País","Contas que registram outros empréstimos e recebíveis cuja contraparte tenha sede ou domicílio no país e que não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Registram-se neste grupo ativos financeiros com pagamentos fixos ou determináveis não cotados em um mercado ativo. Esses títulos são avaliados pelo método de custo amortizado, com os custos de transação capitalizados ao valor do ativo. Instrumentos financeiros representativos da indenização, decorrente da exploração de serviços públicos, constumam receber esta classificação(item 22, OCPC 05).",asset_current,,l10n_br_account_chart_template
account_template_101010550,1.01.01.05.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Valores Mobiliários - No País","Contas que registram os ajustes a valor presente efetuados sobre os ativos financeiros no país. Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do Lalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada (art.4º, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101010555,1.01.01.05.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment)- Valores Mobiliários - No País","Contas que registram as perdas incorridas com base em evidências objetivas passadas, mas posteriores ao reconhecimento inicial, impactantes no fluxo de caixa futuros estimados desses ativos financeiros. Os títulos para negociação não estão sujeitos a teste de “impairment”. Esta conta também registra as eventuais reversões, não se admitindo para títulos patrimoniais. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do Lalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real (art. 32, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101010570,1.01.01.05.70,"Subconta - Ajuste a Valor Justo - Valores Mobiliários – Não Hedge -No País","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os ativos financeiros no país, inclusive decorrentes apenas de sua mensuração inicial ou efetuados nos objetos de hedge de valor justo. Os títulos para negociação e disponíveis para venda devem seguir a mensuração pelo valor justo até sua baixa. Referidos valores deverão ser computados na apuração Lucro Real quando da alienação ou baixa do ativo(arts. 49/50, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101010590,1.01.01.05.90,"Subconta – Adoção Inicial - Valores Mobiliários – Não Hedge - No País","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101010601,1.01.01.06.01,"Derivativos - Hedge Valor Justo - No País","Contas que registram os instrumentos destinados a hedge de valor justo operados em ambiente negocial no país. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",asset_current,,l10n_br_account_chart_template
account_template_101010602,1.01.01.06.02,"Derivativos - Hedge Fluxo de Caixa - No País","Contas que registram os instrumentos destinados a hedge de fluxo de caixa operados em ambiente negocial no país. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",asset_current,,l10n_br_account_chart_template
account_template_101010603,1.01.01.06.03,"Derivativos - Hedge Investimento no Exterior - No País","Contas que registram os instrumentos destinados a hedge de fluxo de investimento no exterior operados em ambiente negocial no país. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",asset_current,,l10n_br_account_chart_template
account_template_101010670,1.01.01.06.70,"Subconta - Ajuste a Valor Justo - Valores Mobiliários – Hedge - No País","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os ativos financeiros no país, destinados a hedge, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da alienação ou baixa do ativo(arts. 51/52, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101010690,1.01.01.06.90,"Subconta – Adoção Inicial - Valores Mobiliários – Hedge - No País","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101010901,1.01.01.09.01,"Títulos para Negociação - Mensurados a Valor Justo por Meio de Resultado (VJPR) - No Exterior","Contas que registram ativos financeiros, cuja contraparte ou ambiente negocial tenham sede ou domicílio no exterior, sobre os quais há a intenção de negociação no curto prazo ou se a mensuração pelo valor justo diminuir ou eliminar alguma inconsistência de mensuração de acordo com a gestão financeira da empresa (fair value option) e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Os derivativos utilizados como hedge devem ser registrados no grupo 1.01.01.10.",asset_current,,l10n_br_account_chart_template
account_template_101010902,1.01.01.09.02,"Títulos Disponíveis para Venda - No Exterior","Contas que registram ativos financeiros, cuja contraparte ou ambiente negocial tenham sede ou domicílio no exterior, sobre os quais não há definição de quando nem quais condições vai negociá-los e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Contrapartida das alterações no seu valor justo, bem como custos de transação, devem ser reconhecidos no Patrimônio Líquido até a realização do ativo.",asset_current,,l10n_br_account_chart_template
account_template_101010903,1.01.01.09.03,"Títulos Mantidos até o Vencimento - No Exterior","Contas que registram ativos financeiros, cuja contraparte ou ambiente negocial tenham sede ou domicílio no exterior, não derivativos, com pagamentos fixos ou determináveis e com vencimento fixo, para os quais há a intenção e a capacidade de se manter até o vencimento e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Esses títulos são avaliados pelo método de custo amortizado, com os custos de transação capitalizados ao valor do ativo. Ações e outros títulos patrimoniais não devem receber esta classificação. Títulos públicos e de renda fixa constumam receber esta classificação.",asset_current,,l10n_br_account_chart_template
account_template_101010910,1.01.01.09.10,"Debêntures emitidas por Partes Relacionadas - No Exterior","Contas que registram as debêntures emitidas por empresas com sede no exterior, relacionadas com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, independente da base de mensuração utilizada ou se tais Valores Mobiliários poderiam ser classificáveis em outras contas mais genéricas de ativos financeiros.",asset_current,,l10n_br_account_chart_template
account_template_101010911,1.01.01.09.11,"Debêntures emitidas por Partes Não Relacionadas - No Exterior","Contas que registram as debêntures emitidas por empresas com sede no exterior, não relacionadas com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, independente da base de mensuração utilizada ou se tais Valores Mobiliários poderiam ser classificáveis em outras contas mais genéricas de ativos financeiros.",asset_current,,l10n_br_account_chart_template
account_template_101010915,1.01.01.09.15,"Outros Empréstimos e Recebíveis - No Exterior","Contas que registram outros empréstimos e recebíveis cuja contraparte tenha sede ou domicílio no exterior e que não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Registram-se neste grupo ativos financeiros com pagamentos fixos ou determináveis não cotados em um mercado ativo. Esses títulos são avaliados pelo método de custo amortizado, com os custos de transação capitalizados ao valor do ativo.",asset_current,,l10n_br_account_chart_template
account_template_101010950,1.01.01.09.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Valores Mobiliários - No Exterior","Contas que registram os ajustes a valor presente efetuados sobre os ativos financeiros no exterior. Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do Lalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada (art.4º, Lei nº 12.973/2014), sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF nº 213/2002.",asset_current,,l10n_br_account_chart_template
account_template_101010955,1.01.01.09.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment)- Valores Mobiliários - No Exterior","Contas que registram as perdas incorridas com base em evidências objetivas passadas, mas posteriores ao reconhecimento inicial, impactantes no fluxo de caixa futuros estimados desses ativos financeiros. Os títulos para negociação não estão sujeitos a teste de “impairment”. Esta conta também registra as eventuais reversões, não se admitindo para títulos patrimoniais. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do Lalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real (art. 32, Lei nº 12.973/2014), sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF nº 213/2002.",asset_current,,l10n_br_account_chart_template
account_template_101010970,1.01.01.09.70,"Subconta - Ajuste a Valor Justo - Não Hedge - No Exterior","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os ativos financeiros no exterior, inclusive decorrentes apenas de sua mensuração inicial ou efetuados nos objetos de hedge de valor justo. Os títulos para negociação e disponíveis para venda devem seguir a mensuração pelo valor justo até sua baixa. Referidos valores deverão ser computados na apuração Lucro Real quando da alienação ou baixa do ativo(arts. 49/50, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6), sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF nº 213/2002. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101010990,1.01.01.09.90,"Subconta – Adoção Inicial - Valores Mobiliários – Não Hedge - No Exterior","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2), sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF nº 213/2002. Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101011001,1.01.01.10.01,"Derivativos - Hedge Valor Justo - No Exterior","Contas que registram os instrumentos destinados a hedge de valor justo operados em ambiente negocial no exterior. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",asset_current,,l10n_br_account_chart_template
account_template_101011002,1.01.01.10.02,"Derivativos - Hedge Fluxo de Caixa - No Exterior","Contas que registram os instrumentos destinados a hedge de fluxo de caixa operados em ambiente negocial no exterior. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",asset_current,,l10n_br_account_chart_template
account_template_101011003,1.01.01.10.03,"Derivativos - Hedge Investimento no Exterior - No Exterior","Contas que registram os instrumentos destinados a hedge de fluxo de investimento no exterior operados em ambiente negocial no exterior. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD",asset_current,,l10n_br_account_chart_template
account_template_101011070,1.01.01.10.70,"Subconta - Ajuste a Valor Justo - Valores Mobiliários – Hedge - No Exterior","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os ativos financeiros no exterior, destinados a Hedge, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da alienação ou baixa do ativo(arts. 49/53, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6) sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF nº 213/2002. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014). Excepcionalmente, as subcontas de adoção inicial deste grupo também devem receber esta classificação no plano referencial.",asset_current,,l10n_br_account_chart_template
account_template_101011090,1.01.01.10.90,"Subconta – Adoção Inicial - Valores Mobiliários – Hedge - No Exterior","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101014001,1.01.01.40.01,"Recursos no Exterior Decorrentes de Exportação","Contas que registram movimentação de recursos em instituições financeiras no exterior oriundas de recebimentos de exportações brasileiras de mercadorias e de serviços.",asset_current,,l10n_br_account_chart_template
account_template_101019901,1.01.01.99.01,"Outras Disponibilidades","Contas que registram outras disponibilidades não classificáveis em contas específicas deste plano de contas.",asset_current,,l10n_br_account_chart_template
account_template_101020101,1.01.02.01.01,"Adiantamentos a Fornecedores - no País – Circulante","Contas que registram os adiantamentos feitos a fornecedores, no país.",asset_current,,l10n_br_account_chart_template
account_template_101020102,1.01.02.01.02,"Adiantamentos a Fornecedores - no Exterior – Circulante","Contas que registram os adiantamentos feitos a fornecedores, no exterior.",asset_current,,l10n_br_account_chart_template
account_template_101020103,1.01.02.01.03,"Adiantamentos a Funcionários – Circulante","Contas que registram os adiantamentos feitos a funcionários.",asset_current,,l10n_br_account_chart_template
account_template_101020104,1.01.02.01.04,"Adiantamentos a Terceiros – Circulante","Contas que registram os adiantamentos feitos a terceiros.",asset_current,,l10n_br_account_chart_template
account_template_101020198,1.01.02.01.98,"Outros Adiantamentos – Circulante","Contas que registram os adiantamentos não classificáveis em contas específicas neste plano de contas.",asset_current,,l10n_br_account_chart_template
account_template_101020201,1.01.02.02.01,"Duplicatas a Receber – Operações com Partes Não Relacionadas - no País","Contas que registram os valores a receber de clientes no país, não relacionados com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, mesmo que haja imediata intenção de venda os títulos. Os recebíveis que foram negociados em processos de securitização, ou assemelhando, devem ser registrados na conta Direitos Creditórios Cedidos(1.01.02.09.20), enquanto não baixados. Já a securitizadora deve registrar os recebíveis em Direitos Creditórios a Receber(1.01.02.09.25)",asset_current,,l10n_br_account_chart_template
account_template_101020202,1.01.02.02.02,"Duplicatas a Receber - Operações com Partes Não Relacionadas - no Exterior","Contas que registram os valores a receber de clientes no exterior, não relacionados com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, mesmo que haja imediata intenção de venda os títulos. Os recebíveis que foram negociados em processos de securitização, ou assemelhando, devem ser registrados na conta Direitos Creditórios Cedidos(1.01.02.09.20), enquanto não baixados. Já a securitizadora deve registrar os recebíveis em Direitos Creditórios a Receber(1.01.02.09.25)",asset_current,,l10n_br_account_chart_template
account_template_101020203,1.01.02.02.03,"Duplicatas a Receber - Operações com Partes Relacionadas - no País","Contas que registram os valores a receber de clientes no país, relacionados com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, mesmo que haja imediata intenção de venda os títulos. Os recebíveis que foram negociados em processos de securitização, ou assemelhando, devem ser registrados na conta Direitos Creditórios Cedidos(1.01.02.09.20), enquanto não baixados. Já a securitizadora deve registrar os recebíveis em Direitos Creditórios a Receber(1.01.02.09.25)",asset_current,,l10n_br_account_chart_template
account_template_101020204,1.01.02.02.04,"Duplicatas a Receber - Operações com Partes Relacionadas - no Exterior","Contas que registram os valores a receber de clientes no exterior, relacionados com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, mesmo que haja imediata intenção de venda os títulos. Os recebíveis que foram negociados em processos de securitização, ou assemelhando, devem ser registrados na conta Direitos Creditórios Cedidos(1.01.02.09.20), enquanto não baixados. Já a securitizadora deve registrar os recebíveis em Direitos Creditórios a Receber(1.01.02.09.25)",asset_current,,l10n_br_account_chart_template
account_template_101020250,1.01.02.02.50,"(-) Juros a Apropriar Decorrentes de Ajuste da Valor Presente (AVP) – Duplicatas a Receber","Contas que registram os ajustes a valor presente efetuados sobre os ativos financeiros representativos das operações de venda à prazo, sua contrapartida não deve ser excluída da Receita Bruta do período, mas compor uma dedução, art. 12, Decreto-Lei nº 1.598/1977(conta referencial 3.01.01.01.02.10). Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do e-Lalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada (art.4º, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101020252,1.01.02.02.52,"(-) Perdas Estimadas em Créditos de Liquidação Duvidosa - Duplicatas a Receber","Contas que registram parcelas a serem subtraídas, correspondentes a valores das perdas estimadas para os créditos de liquidação duvidosa, que retificam este grupo.",asset_current,,l10n_br_account_chart_template
account_template_101020255,1.01.02.02.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Duplicatas a Receber","Contas que registram as perdas incorridas com base em evidências objetivas passadas, mas posteriores ao reconhecimento inicial, impactantes no fluxo de caixa futuros estimados desses ativos financeiros. O registro e dedutibilidade no Lucro Real deve observar os regramentos dispostos nos arts. 24/25, Instrução Normativa SRF nº 1.515/2014",asset_current,,l10n_br_account_chart_template
account_template_101020290,1.01.02.02.90,"Subconta – Adoção Inicial - Duplicatas a Receber","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101020301,1.01.02.03.01,"IPI a Recuperar","Contas que registram o IPI a recuperar.",asset_current,,l10n_br_account_chart_template
account_template_101020302,1.01.02.03.02,"ICMS a Recuperar","Contas que registram o ICMS a recuperar.",asset_current,,l10n_br_account_chart_template
account_template_101020303,1.01.02.03.03,"PIS a Recuperar - Crédito Básico","Contas que registram o PIS a recuperar.",asset_current,,l10n_br_account_chart_template
account_template_101020304,1.01.02.03.04,"PIS a Recuperar - Crédito Presumido","Contas que registram o PIS a recuperar, decorrente de crédito presumido.",asset_current,,l10n_br_account_chart_template
account_template_101020305,1.01.02.03.05,"COFINS a Recuperar - Crédito Básico","Contas que registram a COFINS a recuperar.",asset_current,,l10n_br_account_chart_template
account_template_101020306,1.01.02.03.06,"COFINS a Recuperar - Crédito Presumido","Contas que registram a COFINS a recuperar, decorrente de crédito presumido.",asset_current,,l10n_br_account_chart_template
account_template_101020307,1.01.02.03.07,"CIDE a Recuperar","Contas que registram a CIDE a recuperar.",asset_current,,l10n_br_account_chart_template
account_template_101020340,1.01.02.03.40,"Outros Impostos e Contribuições a Recuperar","Contas que registrem outros impostos e contribuições a recuperar no final do período de apuração. Valores referentes ao ISSQN devem receber esta classificação.",asset_current,,l10n_br_account_chart_template
account_template_101020350,1.01.02.03.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Tributos a Recuperar","Contas que registram os ajustes a valor presente efetuados sobre os tributos a recuperar. Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do Lalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada (art.4º, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101020355,1.01.02.03.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Tributos a Recuperar","Contas que registram as perdas estimadas com base em evidências objetivas impactantes no fluxo de caixa futuros estimados desses tributos a recuperar. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do Lalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real (art. 32, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101020401,1.01.02.04.01,"Imposto de Renda Retido na Fonte (IRRF)","Contas correspondentes ao Imposto de Renda Retido na Fonte sobre receitas da Pessoa Jurídica declarante.",asset_current,,l10n_br_account_chart_template
account_template_101020402,1.01.02.04.02,"IRPJ Recolhido por Estimativa","Contas que registrem o valor do IRPJ recolhido por estimativa.",asset_current,,l10n_br_account_chart_template
account_template_101020403,1.01.02.04.03,"IRPJ Saldo Negativo","Contas que registram o saldo negativo do IRPJ.",asset_current,,l10n_br_account_chart_template
account_template_101020404,1.01.02.04.04,"CSLL Retida na Fonte","Contas correspondentes à Contribuição Social sobre o Lucro Líquido Retida na Fonte sobre receitas da Pessoa Jurídica declarante.",asset_current,,l10n_br_account_chart_template
account_template_101020405,1.01.02.04.05,"CSLL Recolhida por Estimativa","Contas que registram o valor da CSLL recolhida por estimativa.",asset_current,,l10n_br_account_chart_template
account_template_101020406,1.01.02.04.06,"CSLL Saldo Negativo","Contas que registram o saldo negativo da CSLL.",asset_current,,l10n_br_account_chart_template
account_template_101020407,1.01.02.04.07,"PIS/PASEP Retido na Fonte","Contas que registram o PIS/PASEP Retido na Fonte sobre receitas da Pessoa Jurídica declarante.",asset_current,,l10n_br_account_chart_template
account_template_101020408,1.01.02.04.08,"PIS/PASEP a Compensar","Contas que registram o PIS/PASEP a compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020409,1.01.02.04.09,"COFINS Retida na Fonte","Contas que registram a COFINS Retida na Fonte sobre receitas da Pessoa Jurídica declarante.",asset_current,,l10n_br_account_chart_template
account_template_101020410,1.01.02.04.10,"COFINS a Compensar","Contas que registram a COFINS a compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020411,1.01.02.04.11,"IPI a Compensar","Contas que registram ao IPI a compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020412,1.01.02.04.12,"IOF a Compensar","Contas que registram o IOF a compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020413,1.01.02.04.13,"Imposto de Importação a Compensar","Contas que registram o Imposto de Importação a compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020414,1.01.02.04.14,"Imposto de Exportação a Compensar","Contas que registram o Imposto de Exportação a compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020415,1.01.02.04.15,"ITR a Compensar","Contas que registram o ITR a compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020416,1.01.02.04.16,"CIDE a Compensar","Contas que registram a CIDE a compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020417,1.01.02.04.17,"Contribuição Previdenciária Retida na Prestação de Serviços","Contas que registram a Contribuição Previdenciária Retida na Fonte na prestação de serviços a compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020418,1.01.02.04.18,"Contribuição Previdenciária a Compensar","Contas que registram a Contribuição Previdenciáriaa compensar.",asset_current,,l10n_br_account_chart_template
account_template_101020440,1.01.02.04.40,"Outros Tributos a Compensar","Contas que registram outros impostos e contribuições a compensar. Valores referentes ao ISSQN devem receber esta classificação.",asset_current,,l10n_br_account_chart_template
account_template_101020450,1.01.02.04.50,"( - ) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Tributos a Compensar","Contas que registram os ajustes a valor presente efetuados sobre os tributos a compensar. Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do Lalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada(art.4º, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101020455,1.01.02.04.55,"( - ) Perdas por Redução ao Valor Recuperável (Impairment)- Tributos a Compensar","Contas que registram as perdas estimadas com base em evidências objetivas, impactantes no fluxo de caixa futuros estimados desses tributos a compensar. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do Lalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101020901,1.01.02.09.01,"Mútuos com Partes Não Relacionadas – Circulante - No País","Contas que registram empréstimos em moeda efetuados a partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, sediadas no país. Juros e encargos decorrentes também devem ser registrados nesta conta.",asset_current,,l10n_br_account_chart_template
account_template_101020902,1.01.02.09.02,"Mútuos com Partes Não Relacionadas – Circulante - No Exterior","Contas correspondentes a empréstimos em moeda efetuados a partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, sediadas no exterior. Juros e encargos decorrentes também devem ser registrados nesta conta.",asset_current,,l10n_br_account_chart_template
account_template_101020903,1.01.02.09.03,"Dividendos a Receber - Circulante - No País","Contas que registram os dividendos a receber de empresas com sede no país",asset_current,,l10n_br_account_chart_template
account_template_101020904,1.01.02.09.04,"Dividendos a Receber - Circulante - No Exterior","Contas que registram os dividendos a receber de empresas com sede no exterior.",asset_current,,l10n_br_account_chart_template
account_template_101020905,1.01.02.09.05,"Juros Sobre o Capital Próprio a Receber - Circulante","Contas que registram os juros sobre o capital próprio a receber.",asset_current,,l10n_br_account_chart_template
account_template_101020906,1.01.02.09.06,"Adiantamento para Futuro Aumento de Capital –Ativo - Circulante","Contas que registram valores de adiantamento para futuro aumento de capital.",asset_current,,l10n_br_account_chart_template
account_template_101020907,1.01.02.09.07,"Outros Juros a Receber - Circulante","Contas que registram outros juros a receber não classificáveis em contas mais específicas.",asset_current,,l10n_br_account_chart_template
account_template_101020909,1.01.02.09.09,"Contraprestação Contingente Ativa - Combinação de Negócios - Circulante","Contas que registram a contraprestação contingente ativa, em uma combinação de negócios. Em termos gerais, constitui cláusula contratual que confere ao adquirente o direito de reaver parte da contraprestação já transferida, se certas condições específicas venham a ocorrer. Para fins de gerar efeito sobre tratamento fiscal das parcelas integrantes do custo de aquisição de participação societária, deve-se observar os arts. 110/111, Instrução Normativa SRF nº 1.515/2014 .",asset_current,,l10n_br_account_chart_template
account_template_101020910,1.01.02.09.10,"Demais Créditos a Receber - Circulante","Contas que registram os demais créditos a receber, que não possuem conta específica neste plano de contas. Este conta não deve ser utilizada sem esgotamento de outras possibilidades.",asset_current,,l10n_br_account_chart_template
account_template_101020911,1.01.02.09.11,"Depósitos em Contencioso - Circulante","Contas que registram créditos em virtude de depósitos em contencioso de curto prazo.",asset_current,,l10n_br_account_chart_template
account_template_101020912,1.01.02.09.12,"Outros Créditos em Contencioso - Circulante","Contas que registram outros créditos em contencioso de curto prazo.",asset_current,,l10n_br_account_chart_template
account_template_101020920,1.01.02.09.20,"Direitos Creditórios Cedidos","Contas que registram os recebíveis que foram negociados em processos de securitização, ou assemelhantados, enquanto não baixados. Já a Securitizadora deve registrar os recebíveis em Direitos Creditórios a Receber(1.01.02.09.25)",asset_current,,l10n_br_account_chart_template
account_template_101020921,1.01.02.09.21,"(-) Deságio na Cessão de Títulos","Contas que registram o deságio em recebíveis que foram negociados em processos de securitização, ou assemelhantados, enquanto não baixados",asset_current,,l10n_br_account_chart_template
account_template_101020925,1.01.02.09.25,"Direitos Creditórios a Receber","Contas que registram os direitos creditórios adquiridos por empresa que exerça atividade de securitização, ou assemelhantada.",asset_current,,l10n_br_account_chart_template
account_template_101020950,1.01.02.09.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Outros Créditos - Circulante","Contas que registram os ajustes a valor presente efetuados sobre os outros créditos. Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do Lalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada (art.4º, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101020955,1.01.02.09.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Outros Créditos - Circulante","Contas que registram as perdas incorridas com base em evidências objetivas passadas, mas posteriores ao reconhecimento inicial, impactantes no fluxo de caixa futuros estimados desses ativos financeiros. Os títulos para negociação não estão sujeitos a teste de “impairment”. Esta conta também registra as eventuais reversões, não se admitindo para títulos patrimoniais. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do Lalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101020960,1.01.02.09.60,"CPC 47 - Atvos de Contrato - Circulante","Contas que registram os efeitos no ativo circulante, decorrentes da adoção do Pronunciamento Técnico CPC 47 - Receita de Contrato com Cliente.",asset_current,,l10n_br_account_chart_template
account_template_101020970,1.01.02.09.70,"Subconta - Ajuste a Valor Justo – Outros Créditos - Circulante","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os outros créditos, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030101,1.01.03.01.01,"Mercadorias para Revenda","Contas que registram os estoques de mercadorias para revenda.",asset_current,,l10n_br_account_chart_template
account_template_101030155,1.01.03.01.55,"(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Mercadorias","Contas que registram as perdas estimadas em estoques avaliados acima do valor de mercado, ou seja, o excedente ao valor realizável líquido. Devem receber esta classificação contas com tratamento de provisões em perdas no estoque.",asset_current,,l10n_br_account_chart_template
account_template_101030170,1.01.03.01.70,"Subconta - Ajuste a Valor Justo - Estoque Mercadorias","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os estoques objeto de hedge de valor justo ou hedge de fluxo de caixa, quando optado pelo método do “basis adjustments”. Referidos valores deverão ser computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030175,1.01.03.01.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Mercadorias","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de estoques. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda da mercadoria, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030190,1.01.03.01.90,"Subconta – Adoção Inicial - Estoques de Mercadorias","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030201,1.01.03.02.01,"Insumos (materiais diretos)","Contas que registram os estoques de matérias primas e materiais diretos",asset_current,,l10n_br_account_chart_template
account_template_101030202,1.01.03.02.02,"Outros Materiais","Contas que registram estoque de outros materiais.",asset_current,,l10n_br_account_chart_template
account_template_101030203,1.01.03.02.03,"Produtos em Elaboração","Contas que registram os estoques de produtos em elaboração.",asset_current,,l10n_br_account_chart_template
account_template_101030204,1.01.03.02.04,"Produtos Acabados","Contas que registram os estoques de produtos acabados.",asset_current,,l10n_br_account_chart_template
account_template_101030255,1.01.03.02.55,"(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Produtos","Contas que registram as perdas estimadas em estoques avaliados acima do valor de mercado, ou seja, ou seja, o excedente ao valor realizável líquido. Devem receber esta classificação contas com tratamento de provisões em perdas no estoque.",asset_current,,l10n_br_account_chart_template
account_template_101030270,1.01.03.02.70,"Subconta - Ajuste a Valor Justo - Estoque de Produtos","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os estoques objeto de hedge de valor justo ou hedge de fluxo de caixa, quando optado pelo método do “basis adjustments”. Referidos valores deverão ser computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030275,1.01.03.02.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque de Produtos","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de estoques. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até que produto for utilizado na produção de bens ou serviços, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030290,1.01.03.02.90,"Subconta – Adoção Inicial - Estoques de Produtos","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030301,1.01.03.03.01,"Terrenos - Atividade Imobiliária","Contas que registram os terrenos para revenda.",asset_current,,l10n_br_account_chart_template
account_template_101030302,1.01.03.03.02,"Imóveis Adquiridos para Revenda - Atividade Imobiliária","Contas que registram os imóveis adquiridos para revenda.",asset_current,,l10n_br_account_chart_template
account_template_101030303,1.01.03.03.03,"Obras em Andamento - Atividade Imobiliária","Contas que registram as obras em andamento de imóveis para revenda.",asset_current,,l10n_br_account_chart_template
account_template_101030304,1.01.03.03.04,"Imóveis à Venda - Atividade Imobiliária","Contas utilizadas pela pessoa jurídica que exerce atividade imobiliária para indicar o estoque de imóveis destinados à venda existente na data da apuração dos resultados.
Atenção: As construções em andamento de imóveis destinados à venda devem ser incluídas na conta Construções em Andamento de Imóveis Destinados à Venda.",asset_current,,l10n_br_account_chart_template
account_template_101030305,1.01.03.03.05,"Construções em Andamento de Imóveis Destinados à Venda","Contas que registram as construções em andamento de imóveis destinados à venda.",asset_current,,l10n_br_account_chart_template
account_template_101030306,1.01.03.03.06,"Materiais de Construção - Atividade Imobiliária","Contas que registram os materiais de construção relacionados aos imóveis à venda.",asset_current,,l10n_br_account_chart_template
account_template_101030355,1.01.03.03.55,"(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Atividade Imobiliária","Contas que registram as perdas estimadas em estoques avaliados acima do valor de mercado, ou seja, o excedente ao valor realizável líquido. Devem receber esta classificação contas com tratamento de provisões em perdas no estoque.",asset_current,,l10n_br_account_chart_template
account_template_101030375,1.01.03.03.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Atividade Imobiliária","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de estoques. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda ou utilização do estoque na produção, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030390,1.01.03.03.90,"Subconta – Adoção Inicial - Estoques – Atividade Imobiliária","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030401,1.01.03.04.01,"Insumos (materiais diretos) - Estoque Longa Maturação","Contas que registram os estoques de matérias primas e materiais diretos.",asset_current,,l10n_br_account_chart_template
account_template_101030402,1.01.03.04.02,"Outros Materiais - Estoque Longa Maturação","Contas que registram estoques de outros materiais.",asset_current,,l10n_br_account_chart_template
account_template_101030403,1.01.03.04.03,"Produtos em Elaboração - Estoque Longa Maturação","Contas que registram estoques de produtos em elaboração.",asset_current,,l10n_br_account_chart_template
account_template_101030404,1.01.03.04.04,"Produtos Acabados - Estoque Longa Maturação","Contas que registram estoques de produtos acabados.",asset_current,,l10n_br_account_chart_template
account_template_101030455,1.01.03.04.55,"(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Longa Maturação","Contas que registram as perdas estimadas em estoques avaliados acima do valor de mercado, ou seja, o excedente ao valor realizável líquido. Devem receber esta classificação contas com tratamento de provisões em perdas no estoque.",asset_current,,l10n_br_account_chart_template
account_template_101030470,1.01.03.04.70,"Subconta - Ajuste a Valor Justo - Estoque Longa Maturação","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os estoques objeto de hedge de valor justo ou hedge de fluxo de caixa, quando optado pelo método do “basis adjustments”. Referidos valores deverão ser computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030475,1.01.03.04.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Longa Maturação","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de estoques. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até que o produto for utilizado na produção de bens ou serviços, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030490,1.01.03.04.90,"Subconta – Adoção Inicial - Estoques – Longa Maturação","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030501,1.01.03.05.01,"Produtos Agropecuários de Origem Animal","Contas que registram estoques de produtos agropecuários de origem animal. Nos termos do CPC 29, seriam os produtos gerados a partir dos ativos biológicos.",asset_current,,l10n_br_account_chart_template
account_template_101030502,1.01.03.05.02,"Produtos Agropecuários de Origem Vegetal","Contas que registram estoques de produtos agropecuários de origem vegetal. Nos termos do CPC 29, seriam os produtos gerados a partir dos ativos biológicos.",asset_current,,l10n_br_account_chart_template
account_template_101030503,1.01.03.05.03,"Insumos Agropecuários","Contas que registram os estoques de matérias primas e materiais diretos.",asset_current,,l10n_br_account_chart_template
account_template_101030504,1.01.03.05.04,"Outros Materiais - Atividade Rural","Contas que registram os estoques de outros materiais .",asset_current,,l10n_br_account_chart_template
account_template_101030555,1.01.03.05.55,"(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Atividade Rural","Contas que registram as perdas estimadas em estoques avaliados acima do valor de mercado, ou seja, o excedente ao valor realizável líquido. Devem receber esta classificação contas com tratamento de provisões em perdas no estoque.",asset_current,,l10n_br_account_chart_template
account_template_101030570,1.01.03.05.70,"Subconta - Ajuste a Valor Justo - Estoque Atividade Rural","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os estoques objeto de hedge de valor justo ou hedge de fluxo de caixa, quando optado pelo método do “basis adjustments”. Referidos valores deverão ser computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030575,1.01.03.05.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Atividade Rural","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de estoques. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda ou utilização do estoque na produção, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030590,1.01.03.05.90,"Subconta – Adoção Inicial - Estoques – Atividade Rural","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030601,1.01.03.06.01,"Materiais Aplicados na Produção de Serviços","Contas que registram os estoques de materiais aplicados na produção de serviços.",asset_current,,l10n_br_account_chart_template
account_template_101030602,1.01.03.06.02,"Serviços em Andamento","Contas que registram os estoques de serviços em andamento.",asset_current,,l10n_br_account_chart_template
account_template_101030603,1.01.03.06.03,"Serviços Acabados","Contas que registram os estoques de serviços acabados.",asset_current,,l10n_br_account_chart_template
account_template_101030655,1.01.03.06.55,"(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Serviços","Contas que registram as perdas estimadas em estoques avaliados acima do valor de mercado, ou seja, o excedente ao valor realizável líquido. Devem receber esta classificação contas com tratamento de provisões em perdas no estoque.",asset_current,,l10n_br_account_chart_template
account_template_101030670,1.01.03.06.70,"Subconta - Ajuste a Valor Justo - Estoque Serviços","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os estoques objeto de hedge de valor justo ou hedge de fluxo de caixa, quando optado pelo método do “basis adjustments”. Referidos valores deverão ser computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030675,1.01.03.06.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Serviços","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de estoques. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a utilização da mercadoria na produção de serviços, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030690,1.01.03.06.90,"Subconta – Adoção Inicial - Estoque Serviços","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030701,1.01.03.07.01,"Material em Almoxarifado","Contas que estoques de material de almoxarifado, tais como de uso e consumo.",asset_current,,l10n_br_account_chart_template
account_template_101030702,1.01.03.07.02,"Material Destinado à Destruição","Contas que registram estoque de material destinado à destruição.",asset_current,,l10n_br_account_chart_template
account_template_101030703,1.01.03.07.03,"Sucata","Contas que registram estoques de sucata.",asset_current,,l10n_br_account_chart_template
account_template_101030704,1.01.03.07.04,"Outros Estoques","Contas que registram outros estoques que não possuem classificação específica neste plano de contas.",asset_current,,l10n_br_account_chart_template
account_template_101030755,1.01.03.07.55,"(-) Perda por Ajuste ao Valor Realizável Líquido - Estoques Outros","Contas que registram as perdas estimadas em estoques avaliados acima do valor de mercado, ou seja, o excedente ao valor realizável líquido. Devem receber esta classificação contas com tratamento de provisões em perdas no estoque.",asset_current,,l10n_br_account_chart_template
account_template_101030775,1.01.03.07.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Outros","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de estoques. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda ou utilização do estoque na produção, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101030790,1.01.03.07.90,"Subconta – Adoção Inicial - Estoque Outros","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101050101,1.01.05.01.01,"Alugueis Pagos Antecipadamente","Contas que registram os pagamentos antecipados de aluguéis, cujos benefícios à pessoa jurídica ocorrerão durante o exercício seguinte. São valores relativos a despesas que efetivamente pertencem ao exercício seguinte.",asset_prepayments,,l10n_br_account_chart_template
account_template_101050102,1.01.05.01.02,"Prêmios de Seguros a Apropriar","Contas que registram os pagamentos antecipados de prêmios de seguros, cujos benefícios à pessoa jurídica ocorrerão durante o exercício seguinte. São valores relativos a despesas que efetivamente pertencem ao exercício seguinte.",asset_prepayments,,l10n_br_account_chart_template
account_template_101050103,1.01.05.01.03,"Encargos Financeiros a Apropriar","Contas que registram os pagamentos antecipados de despesas financeiras, cujos benefícios à pessoa jurídica ocorrerão durante o exercício seguinte. São valores relativos a despesas que efetivamente pertencem ao exercício seguinte.",asset_prepayments,,l10n_br_account_chart_template
account_template_101050109,1.01.05.01.09,"Outros Custos e Despesas Pagos Antecipadamente","Contas que registram os demais pagamentos antecipados, que não possuem classificação específica neste plano de contas, cujos benefícios ou prestação de serviços à pessoa jurídica ocorrerão durante o exercício seguinte. São valores relativos a despesas que efetivamente pertencem ao exercício seguinte.",asset_prepayments,,l10n_br_account_chart_template
account_template_101100101,1.01.10.01.01,"Ativo Biológico Consumível - Origem Animal – Pelo Valor Justo","Contas que registram, nas empresas com atividade rural, os ativos biológicos consumíveis de origem animal, quando avaliados a valor justo. Nos termos do item 44 do CPC 29, seriam aqueles passíveis de serem colhidos como produto agrícola ou vendidos como ativos biológicos, como rebanhos de animais mantidos para a produção de carne, rebanhos mantidos para a venda, produção de peixe etc. Os produtos agrícolas devem receber classificação de estoques.",asset_current,,l10n_br_account_chart_template
account_template_101100102,1.01.10.01.02,"Ativo Biológico Consumível - Origem Vegetal – Pelo Valor Justo","Contas que registram, nas empresas com atividade rural, os ativos biológicos consumíveis de origem vegetal, quando avaliados a valor justo. Nos termos do item 44 do CPC 29, seriam aqueles passíveis de serem colhidos como produto agrícola ou vendidos como ativos biológicos, como plantações de milho, cana-de-açúcar, soja, árvores para produção de madeira etc. Os produtos agrícolas devem receber classificação de estoques.",asset_current,,l10n_br_account_chart_template
account_template_101100170,1.01.10.01.70,"Subconta - Ajuste a Valor Justo (AVJ) – Ativos Biológicos Consumíveis Pelo Valor Justo - Circulante","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre ativos biológicos consumíveis. Referidos valores deverão ser registrados líquidos da despesa de venda e computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101100175,1.01.10.01.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Ativos Biológicos Pelo Valor Justo - Circulante","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo dos ativos. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda ou baixa do ativo, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF Nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101100190,1.01.10.01.90,"Subconta – Adoção Inicial - Ativo Biologico","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF Nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101100201,1.01.10.02.01,"Ativo Biológico Consumível - Origem Animal - Pelo Custo","Contas que registram, nas empresas com atividade rural, os ativos biológicos consumíveis de origem animal, nos casos em que não seja possível mensurá-los ao valor justo, somente no reconhecimento inicial, devido a indisponibilidade de cotação de mercado e as alternativas não são, claramente, confiáveis. A pessoa jurídica que tenha mensurado previamente o ativo biológico ao seu valor justo, menos a despesa de venda, continuará a mensurá-lo assim até a sua venda. Nos termos do item 44 do CPC 29, seriam aqueles passíveis de serem colhidos como produto agrícola ou vendidos como ativos biológicos, como rebanhos de animais mantidos para a produção de carne, rebanhos mantidos para a venda, produção de peixe etc. Os produtos agrícolas devem receber classificação de estoques.",asset_current,,l10n_br_account_chart_template
account_template_101100202,1.01.10.02.02,"Ativo Biológico Consumível - Origem Vegetal - Pelo Custo","Contas que registram, nas empresas com atividade rural, os ativos biológicos consumíveis de origem animal, nos casos em que não seja possível mensurá-los ao valor justo, somente no reconhecimento inicial, devido a indisponibilidade de cotação de mercado e as alternativas não são, claramente, confiáveis. A pessoa jurídica que tenha mensurado previamente o ativo biológico ao seu valor justo, menos a despesa de venda, continuará a mensurá-lo assim até a sua venda. Nos termos do item 44 do CPC 29, seriam aqueles passíveis de serem colhidos como produto agrícola ou vendidos como ativos biológicos, como plantações de milho, cana-de-açúcar, soja, árvores para produção de madeira etc. Os produtos agrícolas devem receber classificação de estoques.",asset_current,,l10n_br_account_chart_template
account_template_101100255,1.01.10.02.55,"( - ) Perdas por Redução ao Valor Recuperável (Impairment) - Ativos Biológicos Consumível - Pelo Custo","Contas que registram as perdas estimadas com base em evidências objetivas impactantes no fluxo de caixa futuros estimados destes ativos biológicos. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do Lalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101100275,1.01.10.02.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Ativos Biológicos Consumíveis - Pelo Custo","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo dos ativos. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda ou baixa do ativo, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101110101,1.01.11.01.01,"Ativo Não Circulante Mantido Para Venda","Contas que registrem valores de ativos não circulantes mantidos para venda ou destinados a ser distribuídos aos sócios. Nos termos do CPC 31, regra geral, devem ser mensurados pelo menor entre o valor contábil até então registrado e o valor justo menos as despesas de venda/distribuição. Os subsequentes reconhecimentos de depreciações ou amortizações devem cessar enquanto mantidos nesta classificação. Caso ativo ou grupo de ativos for adquirido como parte de combinação de negócios e já receber esta classificação, deve ser mensurado pelo valor justo menos as despesas de venda. Não ocorrendo a venda e não mais presentes os critérios para integrar este grupo, os ativos devem deixá-lo mensurados ao valor mais baixo entre seu valor contábil antes desta classificação, ajustados por qualquer depreciação ou amortização que deixou de ocorrer durante o prazo desta classificação, e seu montante recuperável nesta data.",asset_current,,l10n_br_account_chart_template
account_template_101110155,1.01.11.01.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Ativo Não Circulante Mantido para Venda","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil do ativo ou grupo de ativos. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_current,,l10n_br_account_chart_template
account_template_101110170,1.01.11.01.70,"Subconta - Ajuste a Valor Justo – Ativo Não Circulante Mantido para Venda","Contas que registram os ajustes a valor justo negativos efetuados sobre os ativos não circulantes mantidos para venda, inclusive decorrentes apenas de sua mensuração inicial. Nos acasos em que se apliquem dedução das despesas de vendas ao valor justo, este registro deve ocorrer pelo montante líquido. Referidos valores deverão ser computados na apuração Lucro Real quando da alienação ou baixa do ativo(arts. 41/47, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101110175,1.01.11.01.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Ativo Não Circulante Mantido para Venda","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de estoques. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda da mercadoria, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_101110190,1.01.11.01.90,"Subconta – Adoção Inicial - Ativo Não circulante Mantido para Venda","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_current,,l10n_br_account_chart_template
account_template_102010101,1.02.01.01.01,"Clientes - Longo Prazo","Contas que registram os valores a receber de clientes. Recebe esta classificação no longo prazo, independente se decorrentes de operações com ou não partes relacionadas, no país ou exterior.",asset_non_current,,l10n_br_account_chart_template
account_template_102010102,1.02.01.01.02,"Mútuos com Partes Não Relacionadas - Ativo- Longo Prazo","Contas que registram a empréstimos efetuados a partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, seja sediada no país ou exterior. Juros e encargos decorrentes também devem ser registrados nesta conta.",asset_non_current,,l10n_br_account_chart_template
account_template_102010103,1.02.01.01.03,"Mútuos com Partes Relacionadas - Ativo - Longo Prazo","Contas que registram empréstimos efetuados a partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, seja sediada no país ou exterior. Juros e encargos decorrentes também devem ser registrados nesta conta.",asset_non_current,,l10n_br_account_chart_template
account_template_102010104,1.02.01.01.04,"Adiantamento para Futuro Aumento de Capital - Ativo - Longo Prazo","Contas que registram valores de adiantamento para futuro aumento de capital.",asset_non_current,,l10n_br_account_chart_template
account_template_102010107,1.02.01.01.07,"Títulos Patrimoniais Avaliados pelo Custo - Longo Prazo","Contas que registram os títulos patrimoniais, participações societárias, avaliados pelo custo de aquisição, quando o valor justo não puder ser aplicado por ausência de informações confiáveis.",asset_non_current,,l10n_br_account_chart_template
account_template_102010109,1.02.01.01.09,"Contraprestação Contingente Ativa - Combinação de Negócios – Longo Prazo","Contas que registram a contraprestação contingente ativa, em uma combinação de negócios. Em termos gerais, constitui cláusula contratual que confere ao adquirente o direito de reaver parte da contraprestação já transferida, se certas condições específicas venham a ocorrer. Para fins de gerar efeito sobre tratamento fiscal das parcelas integrantes do custo de aquisição de participação societária, deve-se observar os arts. 110/111, Instrução Normativa SRF nº 1.515/2014 .",asset_non_current,,l10n_br_account_chart_template
account_template_102010110,1.02.01.01.10,"Outros Valores Mobiliários - No Exterior - Longo Prazo","Contas que registram os créditos a receber no exterior, que não possuem classificação adequada no grupo 1.02.01.03. Na data de migração do valor para o Circulante deve-se observar se há conta específica para melhor classificação.",asset_non_current,,l10n_br_account_chart_template
account_template_102010119,1.02.01.01.19,"(-) Outras Contas Retificadoras – Créditos e Valores - Longo Prazo","Contas que registram parcelas a serem subtraídas do Ativo Não Circulante que não possam ser classificadas em outras contas desse plano de contas.",asset_non_current,,l10n_br_account_chart_template
account_template_102010120,1.02.01.01.20,"Direitos Creditórios Cedidos –Longo Prazo","Contas que registram os recebíveis que foram negociados em processos de securitização, ou assemelhando, enquanto não baixados. Já a securitizadora deve registrar os recebíveis em Direitos Creditórios a Receber(1.01.02.09.25)",asset_non_current,,l10n_br_account_chart_template
account_template_102010121,1.02.01.01.21,"(-) Deságio na Cessão de Títulos–Longo Prazo","Contas que registram o deságio em recebíveis que foram negociados em processos de securitização, ou assemelhando, enquanto não baixados.",asset_non_current,,l10n_br_account_chart_template
account_template_102010125,1.02.01.01.25,"Direitos Creditórios a Receber–Longo Prazo","Contas que registram os direitos creditórios adquiridos por empresa que exerça atividade de securitização ou assemelhantada.",asset_non_current,,l10n_br_account_chart_template
account_template_102010126,1.02.01.01.26,"Duplicatas a Receber – Operações com Partes Não Relacionadas - no País – Longo Prazo","Contas que registram os valores a receber de clientes no país de longo prazo, não relacionados com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, mesmo que haja imediata intenção de venda os títulos.",asset_non_current,,l10n_br_account_chart_template
account_template_102010127,1.02.01.01.27,"Duplicatas a Receber - Operações com Partes Não Relacionadas - no Exterior – Longo Prazo","Contas que registram os valores a receber de clientes no exterior de longo prazo, não relacionados com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, mesmo que haja imediata intenção de venda os títulos.",asset_non_current,,l10n_br_account_chart_template
account_template_102010128,1.02.01.01.28,"Duplicatas a Receber - Operações com Partes Relacionadas - no País – Longo Prazo","Contas que registram os valores a receber de clientes no país de longo prazo, relacionados com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, mesmo que haja imediata intenção de venda os títulos.",asset_non_current,,l10n_br_account_chart_template
account_template_102010129,1.02.01.01.29,"Duplicatas a Receber - Operações com Partes Relacionadas - no Exterior – Longo Prazo","Contas que registram os valores a receber de clientes no exterior de longo prazo, relacionados com a declarante conforme conceito definido no CPC 05(R1), itens 09 a 12, mesmo que haja imediata intenção de venda os títulos.",asset_non_current,,l10n_br_account_chart_template
account_template_102010150,1.02.01.01.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Créditos –Longo Prazo","Contas que registram os ajustes a valor presente efetuados sobre os créditos a longo prazo, sua contrapartida não deve ser excluída da Receita Bruta do período, mas compor uma dedução, art. 12, Decreto-Lei nº 1.598/1977(conta referencial 3.01.01.01.02.10). Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do e-Lalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada (art.4º, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102010152,1.02.01.01.52,"(-) Perdas Estimadas em Créditos de Liquidação Duvidosa – Longo Prazo","Contas que registram parcelas a serem subtraídas, correspondentes a valores das perdas estimadas para os créditos de liquidação duvidosa.",asset_non_current,,l10n_br_account_chart_template
account_template_102010155,1.02.01.01.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Créditos e Valores - Longo Prazo","Contas que registram as perdas incorridas com base em evidências objetivas passadas, mas posteriores ao reconhecimento inicial, impactantes no fluxo de caixa futuros estimados desses ativos financeiros. O registro e dedutibilidade no Lucro Real deve observar os regramentos dispostos nos arts. 24/25, Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102010170,1.02.01.01.70,"Subconta - Ajuste a Valor Justo - Créditos –Longo Prazo","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os ativos financeiros, inclusive decorrentes apenas de sua mensuração inicial. . Referidos valores deverão ser computados na apuração Lucro Real quando da alienação ou baixa do ativo(arts. 49/50, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102010190,1.02.01.01.90,"Subconta – Adoção Inicial - Créditos e Valores – Longo Prazo","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102010201,1.02.01.02.01,"Títulos para Negociação - No País - Longo Prazo","Contas que registram ativos financeiros, utilizados em operações de hedge ou não, cuja contraparte ou ambiente negocial tenham sede ou domicílio no país, sobre os quais há a intenção de negociação no curto prazo ou se a mensuração pelo valor justo diminuir ou eliminar alguma inconsistência de mensuração de acordo com a gestão financeira da empresa (fair value option) e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76",asset_non_current,,l10n_br_account_chart_template
account_template_102010202,1.02.01.02.02,"Títulos Disponíveis para Venda - No País - Longo Prazo","Contas que registram ativos financeiros, utilizados em operações de hedge ou não, cuja contraparte ou ambiente negocial tenham sede ou domicílio no país, sobre os quais não há definição de quando nem quais condições vai negociá-los e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Contrapartida das alterações no seu valor justo, bem como custos de transação, devem ser reconhecidos no Patrimônio Líquido até a realização do ativo.",asset_non_current,,l10n_br_account_chart_template
account_template_102010203,1.02.01.02.03,"Títulos Mantidos até o Vencimento - No País - Longo Prazo","Contas que registram ativos financeiros, cuja contraparte ou ambiente negocial tenham sede ou domicílio no país, com pagamentos fixos ou determináveis e com vencimento fixo, para os quais há a intenção e a capacidade de se manter até o vencimento e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Esses títulos são avaliados pelo método de custo amortizado, com os custos de transação capitalizados ao valor do ativo. Ações e outros títulos patrimoniais não devem receber esta classificação. Títulos públicos e de renda fixa constumam receber esta classificação.",asset_non_current,,l10n_br_account_chart_template
account_template_102010210,1.02.01.02.10,"Debêntures emitidas por Partes Relacionada - No País - Longo Prazo","Contas que registram as debêntures de longo prazo emitidas por empresas com sede no país, relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12. Independente da base de mensuração utilizada ou se tais Valores Mobiliários poderiam ser classificáveis em outras contas mais genéricas de ativos financeiros.",asset_non_current,,l10n_br_account_chart_template
account_template_102010211,1.02.01.02.11,"Debêntures emitidas por Partes Não Relacionada - No País - Longo Prazo","Contas que registram as debêntures de longo prazo emitidas por empresas com sede no país, não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12. Independente da base de mensuração utilizada ou se tais Valores Mobiliários poderiam ser classificáveis em outras contas mais genéricas de ativos financeiros.",asset_non_current,,l10n_br_account_chart_template
account_template_102010214,1.02.01.02.14,"Outros Valores Mobiliários – No País - Longo Prazo","Contas que registram outros valores mobiliários de longo prazo, cuja contraparte tenha sede ou domicílio no país, e que não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76.",asset_non_current,,l10n_br_account_chart_template
account_template_102010215,1.02.01.02.15,"Outros Empréstimos e Recebíveis – No País - Longo Prazo","Contas que registram outros empréstimos e recebíveis de longo prazo, cuja contraparte tenha sede ou domicílio no país, e que não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Registram-se neste grupo ativos financeiros com pagamentos fixos ou determináveis não cotados em um mercado ativo. Esses títulos são avaliados pelo método de custo amortizado, com os custos de transação capitalizados ao valor do ativo. Instrumentos financeiros representativos da indenização, decorrente da exploração de serviços públicos, constumam receber esta classificação(item 22, OCPC05).",asset_non_current,,l10n_br_account_chart_template
account_template_102010216,1.02.01.02.16,"Outros Juros a Receber – No País - Longo Prazo","Contas que registram outros juros a receber de longo prazo, cuja contraparte tenha sede ou domicílio no país, e que não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76.",asset_non_current,,l10n_br_account_chart_template
account_template_102010250,1.02.01.02.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) Valores Mobiliários – No País - Longo Prazo","Contas que registram os ajustes a valor presente efetuados sobre os créditos a longo prazo. Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do Lalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada (art.4º, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102010255,1.02.01.02.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Valores Mobiliários - No País - Longo Prazo","Contas que registram as perdas incorridas com base em evidências objetivas passadas, mas posteriores ao reconhecimento inicial, impactantes no fluxo de caixa futuros estimados desses ativos financeiros. Os títulos para negociação não estão sujeitos a teste de “impairment”. Esta conta também registra as eventuais reversões, não se admitindo para títulos patrimoniais. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do Lalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real (art. 32, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102010270,1.02.01.02.70,"Subconta - Ajuste a Valor Justo – Valores Mobiliários – No País - Longo Prazo","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os ativos financeiros no país, inclusive decorrentes apenas de sua mensuração inicial ou efetuados nos objetos de hedge de valor justo. Os títulos para negociação e disponíveis para venda devem seguir a mensuração pelo valor justo até sua baixa. Referidos valores deverão ser computados na apuração Lucro Real quando da alienação ou baixa do ativo(arts. 49/53, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102010290,1.02.01.02.90,"Subconta – Adoção Inicial - Valores Mobiliários - No País – Longo Prazo","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102010301,1.02.01.03.01,"Títulos para Negociação - No Exterior - Longo Prazo","Contas que registram ativos financeiros, utilizados em operações de hedge ou não, cuja contraparte ou ambiente negocial tenham sede ou domicílio no exterior, sobre os quais há a intenção de negociação no curto prazo ou se a mensuração pelo valor justo diminuir ou eliminar alguma inconsistência de mensuração de acordo com a gestão financeira da empresa (fair value option) e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76.",asset_non_current,,l10n_br_account_chart_template
account_template_102010302,1.02.01.03.02,"Títulos Disponíveis para Venda - No Exterior - Longo Prazo","Contas que registram ativos financeiros, utilizados em operações de hedge ou não, cuja contraparte ou ambiente negocial tenham sede ou domicílio no exterior, sobre os quais não há definição de quando nem quais condições vai negociá-los e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Contrapartida das alterações no seu valor justo, bem como custos de transação, devem ser reconhecidos no Patrimônio Líquido até a realização do ativo.",asset_non_current,,l10n_br_account_chart_template
account_template_102010303,1.02.01.03.03,"Títulos Mantidos até o Vencimento - No Exterior - Longo Prazo","Contas que registram ativos financeiros, cuja contraparte ou ambiente negocial tenham sede ou domicílio no exterior, com pagamentos fixos ou determináveis e com vencimento fixo, para os quais há a intenção e a capacidade de se manter até o vencimento e ainda não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Esses títulos são avaliados pelo método de custo amortizado, com os custos de transação capitalizados ao valor do ativo. Ações e outros títulos patrimoniais não devem receber esta classificação. Títulos públicos e de renda fixa constumam receber esta classificação.",asset_non_current,,l10n_br_account_chart_template
account_template_102010304,1.02.01.03.04,"Debêntures emitidas por Partes Relacionada – No Exterior - Longo Prazo","Contas que registram as debêntures de longo prazo emitidas por empresas com sede no exterior, relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12. Independente da base de mensuração utilizada ou se tais Valores Mobiliários poderiam ser classificáveis em outras contas mais genéricas de ativos financeiros.",asset_non_current,,l10n_br_account_chart_template
account_template_102010305,1.02.01.03.05,"Debêntures emitidas por Partes Não Relacionada - No Exterior - Longo Prazo","Contas que registram as debêntures de longo prazo emitidas por empresas com sede no exterior, não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12. Independente da base de mensuração utilizada ou se tais Valores Mobiliários poderiam ser classificáveis em outras contas mais genéricas de ativos financeiros.",asset_non_current,,l10n_br_account_chart_template
account_template_102010306,1.02.01.03.06,"Outros Empréstimos e Recebíveis – No Exterior - Longo Prazo","Contas que registram outros empréstimos e recebíveis de longo prazo, cuja contraparte tenha sede ou domicílio no exterior, e que não estejam melhor classificados em outras contas mais específica, mesmo que extrapolem o conceito da Lei nº 6.385/76. Registram-se neste grupo ativos financeiros com pagamentos fixos ou determináveis não cotados em um mercado ativo. Esses títulos são avaliados pelo método de custo amortizado, com os custos de transação capitalizados ao valor do ativo. Instrumentos financeiros representativos da indenização, decorrente da exploração de serviços públicos, constumam receber esta classificação(item 22, OCPC05).",asset_non_current,,l10n_br_account_chart_template
account_template_102010350,1.02.01.03.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) Valores Mobiliários – No Exterior - Longo Prazo","Contas que registram os ajustes a valor presente efetuados sobre os créditos a longo prazo. Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do Lalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada (art.4º, Lei nº 12.973/2014), sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF nº 213/2002.",asset_non_current,,l10n_br_account_chart_template
account_template_102010355,1.02.01.03.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Valores Mobiliário - No Exterior - Longo Prazo","Contas que registram as perdas incorridas com base em evidências objetivas passadas, mas posteriores ao reconhecimento inicial, impactantes no fluxo de caixa futuros estimados desses ativos financeiros. Os títulos para negociação não estão sujeitos a teste de “impairment”. Esta conta também registra as eventuais reversões, não se admitindo para títulos patrimoniais. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do Lalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real (art. 32, Lei nº 12.973/2014), sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF nº 213/2002.",asset_non_current,,l10n_br_account_chart_template
account_template_102010370,1.02.01.03.70,"Subconta - Ajuste a Valor Justo – Valores Mobiliários – No Exterior - Longo Prazo","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os ativos financeiros, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da alienação ou baixa do ativo(arts. 49/53, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6) sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF nº 213/2002. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102010390,1.02.01.03.90,"Subconta – Adoção Inicial - Valores Mobiliários - No Exterior – Longo Prazo","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2), sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF nº 213/2002. Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102010501,1.02.01.05.01,"Créditos Fiscais CSLL - Diferenças Temporárias e Base de Cálculo Negativa - Longo Prazo","Contas que registram o valor do tributo recuperável em período futuro relacionado a diferenças temporárias dedutíveis, compensação de bases de cálculo negativas de CSLL e compensação de créditos fiscais. Diferenças temporárias são divergências no valor contábil de ativo ou passivo no balanço e sua base fiscal, nos termos do CPC32.",asset_non_current,,l10n_br_account_chart_template
account_template_102010502,1.02.01.05.02,"Créditos Fiscais IRPJ - Diferenças Temporárias e Prejuízos Fiscais - Longo Prazo","Contas que registram o valor do tributo recuperável em período futuro relacionado a diferenças temporárias dedutíveis, compensação de prejuízos ficais e compensação de créditos fiscais. Diferenças temporárias são divergências no valor contábil de ativo ou passivo no balanço e sua base fiscal, nos termos do CPC32.",asset_non_current,,l10n_br_account_chart_template
account_template_102010701,1.02.01.07.01,"Depósitos em Contencioso - Longo Prazo","Contas que registram os depósitos efetuados, decorrentes de demanda contenciosa, a qualquer título, pendentes de decisão, que se realizarão em período posterior ao exercício seguinte à data do balanço.",asset_non_current,,l10n_br_account_chart_template
account_template_102010710,1.02.01.07.10,"Outros Créditos em Contencioso - Longo Prazo","Contas que registram créditos decorrentes de demanda contenciosa, de qualquer natureza, pendentes de decisão, que se realizarão em período posterior ao exercício seguinte à data do balanço.",asset_non_current,,l10n_br_account_chart_template
account_template_102010755,1.02.01.07.55,"( - ) Perdas por Redução ao Valor Recuperável (Impairment) - Créditos em Contencioso – Longo Prazo","Contas que registram as perdas estimadas com base em evidências objetivas impactantes no fluxo de caixa futuros estimados desses créditos em contecioso. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102010801,1.02.01.08.01,"IPI a Recuperar - Longo Prazo","Contas que registram o IPI a recuperar no longo prazo.",asset_non_current,,l10n_br_account_chart_template
account_template_102010802,1.02.01.08.02,"ICMS a Recuperar - Longo Prazo","Contas que registram o ICMS a recuperar no longo prazo.",asset_non_current,,l10n_br_account_chart_template
account_template_102010803,1.02.01.08.03,"PIS a Recuperar - Crédito Básico - Longo Prazo","Contas correspondentes ao PIS a recuperar após o final do período de apuração seguinte (longo prazo).",asset_non_current,,l10n_br_account_chart_template
account_template_102010804,1.02.01.08.04,"PIS a Recuperar - Crédito Presumido - Longo Prazo","Contas que registram o PIS a recuperar no longo prazo, decorrente de crédito presumido.",asset_non_current,,l10n_br_account_chart_template
account_template_102010805,1.02.01.08.05,"COFINS a Recuperar - Crédito Básico - Longo Prazo","Contas que registram a COFINS a recuperar no longo prazo.",asset_non_current,,l10n_br_account_chart_template
account_template_102010806,1.02.01.08.06,"COFINS a Recuperar - Crédito Presumido - Longo Prazo","Contas que registram a COFINS a recuperar no longo prazo, decorrente de crédito presumido.",asset_non_current,,l10n_br_account_chart_template
account_template_102010807,1.02.01.08.07,"CIDE a Recuperar - Longo Prazo","Contas que registram a CIDE a recuperar no longo prazo.",asset_non_current,,l10n_br_account_chart_template
account_template_102010840,1.02.01.08.40,"Outros Impostos e Contribuições a Recuperar - Longo Prazo","Contas que registram outros impostos e contribuições a recuperar no longo prazo. Valores referentes ao ISSQN devem receber esta classificação.",asset_non_current,,l10n_br_account_chart_template
account_template_102010850,1.02.01.08.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Tributos a Recuperar - Longo Prazo","Contas que registram os ajustes a valor presente efetuados sobre os tributos a recuperar. Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do eLalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada (art.4º, Lei 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102010855,1.02.01.08.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Tributos a Recuperar - Longo Prazo","Contas que registram as perdas estimadas com base em evidências objetivas impactantes no fluxo de caixa futuros estimados desses tributos a recuperar. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real (art. 32, Lei 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102010901,1.02.01.09.01,"Alugueis pagos Antecipadamente - Longo Prazo","Contas que registram pagamentos antecipados de aluguéis, cujos benefícios à pessoa jurídica ocorrerão em período posterior ao exercício seguinte à data do balanço. São valores relativos a despesas que efetivamente pertencem a período posterior ao exercício seguinte à data do balanço.",asset_non_current,,l10n_br_account_chart_template
account_template_102010902,1.02.01.09.02,"Prêmios de Seguros a Apropriar - Longo Prazo","Contas que registram pagamentos antecipados de prêmios de seguros, cujos benefícios à pessoa jurídica ocorrerão durante o exercício seguinte. São valores relativos a despesas que efetivamente pertencem a período posterior ao exercício seguinte à data do balanço.",asset_non_current,,l10n_br_account_chart_template
account_template_102010903,1.02.01.09.03,"Encargos Financeiros a Apropriar –Longo Prazo","Contas que registram pagamentos antecipados de despesas financeiras, cujos benefícios à pessoa jurídica ocorrerão após término do exercício seguinte. São valores relativos a despesas que efetivamente pertencem a períodos posteriores ao exercício seguinte.",asset_non_current,,l10n_br_account_chart_template
account_template_102010909,1.02.01.09.09,"Outros Custos e Despesas Pagos Antecipadamente - Longo Prazo","Contas que registram demais pagamentos antecipados, que não possuem classificação específica neste plano de contas, cujos benefícios ou prestação de serviços à pessoa jurídica ocorrerão durante o exercício seguinte. São valores relativos a despesas que efetivamente pertencem a período posterior ao exercício seguinte à data do balanço.",asset_non_current,,l10n_br_account_chart_template
account_template_102011001,1.02.01.10.01,"Ativo Biológico Consumível - Origem Animal – Pelo Valor Justo - Longo Prazo","Contas que registram, nas empresas com atividade rural, os ativos biológicos consumíveis de origem animal, quando avaliados a valor justo. Nos termos do item 44 do CPC 29, seriam aqueles passíveis de serem colhidos como produto agrícola ou vendidos como ativos biológicos, como rebanhos de animais mantidos para a produção de carne, rebanhos mantidos para a venda, produção de peixe etc. Os produtos agrícolas devem receber classificação de estoques.",asset_non_current,,l10n_br_account_chart_template
account_template_102011002,1.02.01.10.02,"Ativo Biológico Consumível - Origem Vegetal – Pelo Valor Justo - Longo Prazo","Contas que registram, nas empresas com atividade rural, os ativos biológicos consumíveis de origem vegetal, quando avaliados a valor justo. Nos termos do item 44 do CPC 29, seriam aqueles passíveis de ser vendidos como ativos biológicos, como plantações de milho, cana-de-açúcar, soja, árvores para produção de madeira etc. Os produtos agrícolas devem receber classificação de estoques.",asset_non_current,,l10n_br_account_chart_template
account_template_102011010,1.02.01.10.10,"Ativo Biológico Consumível - Origem Animal - Pelo Custo - Longo Prazo","Contas que registram, nas empresas com atividade rural, os ativos biológicos consumíveis de origem animal, nos casos em que não seja possível mensurá-los ao valor justo, somente no reconhecimento inicial, devido a indisponibilidade de cotação de mercado e as alternativas não são, claramente, confiáveis. A pessoa jurídica que tenha mensurado previamente o ativo biológico ao seu valor justo, menos a despesa de venda, continuará a mensurá-lo assim até a sua venda. Nos termos do item 44 do CPC 29, seriam aqueles passíveis de serem colhidos como produto agrícola ou vendidos como ativos biológicos, como rebanhos de animais mantidos para a produção de carne, rebanhos mantidos para a venda, produção de peixe etc. Os produtos agrícolas devem receber classificação de estoques.",asset_non_current,,l10n_br_account_chart_template
account_template_102011011,1.02.01.10.11,"Ativo Biológico Consumível - Origem Vegetal - Pelo Custo - Longo Prazo","Contas que registram, nas empresas com atividade rural, os ativos biológicos consumíveis de origem animal, nos casos em que não seja possível mensurá-los ao valor justo, somente no reconhecimento inicial, devido a indisponibilidade de cotação de mercado e as alternativas não são, claramente, confiáveis. A pessoa jurídica que tenha mensurado previamente o ativo biológico ao seu valor justo, menos a despesa de venda, continuará a mensurá-lo assim até a sua venda. Nos termos do item 44 do CPC 29, seriam aqueles passíveis de serem colhidos como produto agrícola ou vendidos como ativos biológicos, como plantações de milho, cana-de-açúcar, soja, árvores para produção de madeira etc. Os produtos agrícolas devem receber classificação de estoques.",asset_non_current,,l10n_br_account_chart_template
account_template_102011055,1.02.01.10.55,"( - ) Perdas por Redução ao Valor Recuperável (Impairment) - Ativos Biológicos Consumível – Longo Prazo","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil destes ativos biológicos. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102011070,1.02.01.10.70,"Subconta - Ajuste a Valor Justo (AVJ) – Ativos Biológicos - Longo Prazo","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre ativos biológicos consumíveis. Referidos valores deverão ser registrados líquidos da despesa de venda e computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102011075,1.02.01.10.75,"Subconta – Ajuste a Valor Presente (AVP) - Ativos Biológicos - Longo Prazo","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo dos ativos. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda ou baixa do ativo, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102011090,1.02.01.10.90,"Subconta – Adoção Inicial – Ativo Biológico – Longo Prazo","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102011501,1.02.01.15.01,"Outros Créditos - Longo Prazo","Contas que registram outros créditos de longo prazo não classificados em contas mais específicas.",asset_non_current,,l10n_br_account_chart_template
account_template_102011560,1.02.01.15.60,"CPC 47 - Atvos de Contrato - Longo Prazo","Contas que registram os efeitos no ativo não circulante, decorrentes da adoção do Pronunciamento Técnico CPC 47 - Receita de Contrato com Cliente.",asset_non_current,,l10n_br_account_chart_template
account_template_102020101,1.02.02.01.01,"Participações Permanentes em Controladas - no País","Contas que registram investimentos permanentes no país, na forma de participação em outras sociedades nas quais se detenham o controle. Exceto sociedades de propósito específico. Vale ressaltar que as participações societárias serão avaliadas, na adoção inicial, conforme regra vigente na Lei nº 6.404/1976(art. 64, Lei nº 12.973/2014), não gerando ocorrências de controle em subcontas de adoção inicial para todas as parcelas integrantes do custo de aquisição(art. 173, §2º, Instrução Normativa SRF nº 1515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020104,1.02.02.01.04,"Participações Permanentes em Coligadas - no País","Contas que registram investimentos permanentes no país, na forma de participação em outras sociedades nas quais se tenha influência significativa. Exceto em sociedades de propósito específico. Vale ressaltar que as participações societárias serão avaliadas, na adoção inicial, conforme regra vigente na Lei nº 6.404/1976(art. 64, Lei nº 12.973/2014), não gerando ocorrências de controle em subcontas de adoção inicial para todas as parcelas integrantes do custo de aquisição(art. 173, §2º, Instrução Normativa SRF nº 1515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020105,1.02.02.01.05,"Participações Permanentes em Joint Ventures e Sociedades de Propósito Específico (SPE) - no País","Contas que registram investimentos permanentes no país, na forma de participação permanente em joint ventures e sociedades de propósito específico, avaliadas pelo Método de Equivalência Patrimonial, ainda que as investidas sejam controladas ou coligadas. Vale ressaltar que as participações societárias serão avaliadas, na adoção inicial, conforme regra vigente na Lei nº 6.404/1976(art. 64, Lei nº 12.973/2014), não gerando ocorrências de controle em subcontas de adoção inicial para todas as parcelas integrantes do custo de aquisição(art. 173, §2º, Instrução Normativa SRF nº 1515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020106,1.02.02.01.06,"Participações Permanentes em Outras Sociedades do Mesmo Grupo ou Controle Comum - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no País","Contas que registram investimentos permanentes no país, na forma de participação permanente em outras sociedades do grupo ou controle comum, sem que haja relação de controle ou influência significativa, avaliados pelo Método de Equivalência Patrimonial(art. 248, Lei nº 6.404/1976). Vale ressaltar que as participações societárias serão avaliadas, na adoção inicial, conforme regra vigente na Lei nº 6.404/1976(art. 64, Lei nº 12.973/2014), não gerando ocorrências de controle em subcontas de adoção inicial para todas as parcelas integrantes do custo de aquisição(art. 173, §2º, Instrução Normativa SRF nº 1515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020107,1.02.02.01.07,"Participações em Sociedades em Conta de Participação (SCP) - Sócio Ostensivo - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no País","Contas que registram investimentos permanentes no país, na forma de participação em sociedades por conta de participação pelo sócio ostensivo, avaliados pelo Método da Equivalência Patrimonial. Vale ressaltar que as participações societárias serão avaliadas, na adoção inicial, conforme regra vigente na Lei nº 6.404/1976(art. 64, Lei nº 12.973/2014), não gerando ocorrências de controle em subcontas de adoção inicial para todas as parcelas integrantes do custo de aquisição(art. 173, §2º, Instrução Normativa SRF nº 1515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020108,1.02.02.01.08,"Participações em Sociedades em Conta de Participação (SCP) - Sócio Participante - no País","Contas que registram investimentos permanentes no país, na forma de participação em sociedades por conta de participação pelo sócio participante, avaliados pelo Método da Equivalência Patrimonial. Vale ressaltar que as participações societárias serão avaliadas, na adoção inicial, conforme regra vigente na Lei nº 6.404/1976(art. 64, Lei nº 12.973/2014), não gerando ocorrências de controle em subcontas de adoção inicial para todas as parcelas integrantes do custo de aquisição(art. 173, §2º, Instrução Normativa SRF nº 1515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020110,1.02.02.01.10,"Goodwill em Investimentos - no País","Contas que registram o ágio por expectativa de rentabilidade futura, referente apenas a aquisições de participações societárias no país, apurado segundo regramentos societários, e tratado no art. 20, inc.III, Decreto-Lei nº 1.598/1977. Por não possuírem prazo definido, não estão sujeitos à amortização societária, apenas redução por impairment, cujo valor será adicionado ao Lucro Real do período(art.25, Lei nº 12.973/2014) e posteriormente excluído, quando da alienação da participação, para fins de cálculo do ganho ou perda de capital(art. 33, inc. II, Decreto-Lei nº 1.598/1977 ). O saldo original apurado na data da aquisição societária deve ser registrado na Parte B do eLalur, somente podendo vir a ser excluído do Lucro Real em parcelas mensais, nos termos do art. 22 da Lei nº 12.973/2014. Esta conta não registra goodwill gerado em outras modalidades de combinações de negócio previsas no CPC 15, bem como, em eventos societários de incorporação, fusão e cisão, cujos valores devem ser registrados em conta de intangível(1.02.05.01.20). Merecem especial atenção os valores de ágio gerados em aquisições de participação societárias no país, ocorridas ou iniciadas até 31/12/2014, caso ainda calculado pela metodologia preceitua na redação original do art. 20, Decreto-Lei nº 1.598/1977, não mais vigente, contudo, aplicável segundo regramento transitório previsto no art. 65, Lei 12.973/2014. Estes valores devem ser controlados apenas extracontabilmente em memória de cálculo encaminhada para ECF no registro Y800(art. 107, da Instrução Normativa SRF nº 1.515/2014). Caso as condições necessárias para início da amortização fiscal via eLalur não venham a se materializar nos termos do art.106 da mesma Instrução Normativa, eventual goodwill registrado contabilmente deve seguir regra aplicável aos demais casos.",asset_non_current,,l10n_br_account_chart_template
account_template_102020111,1.02.02.01.11,"Mais Valia em Investimentos - no País","Contas que registram a mais-valia, referente a aquisições de participações societárias no país, apurada segundo regramentos societários, e tratada no art. 20, inc.II, Decreto-Lei nº 1.598/1977. A amortização societária ocorrerá, conforme bem ou direito for realizado na investida, devendo valor ser adicionado ao Lucro Real do período(art. 25, Decreto-Lei nº 1.598/1977) e posteriormente excluído, quando da alienação da participação, para fins de cálculo do ganho ou perda de capital(art. 33, inc. II, Decreto-Lei nº 1.598/1977 ). O saldo original apurado na data da aquisição societária deve ser registrado na Parte B do eLalur, somente virndo a ser considerado integrante do custo do bem ou direito que lhe deu causa, ou ser excluído do Lucro Real em parcelas mensais, nos termos do art. 20 da Lei nº 12.973/2014. No caso do referido bem ou direito e a mais-valia correlata fundirem-se em um mesmo patrimônio, o saldo da mais-valia registrado na investidora na data do evento societário(p.ex. incorporação) integrará o custo do bem ou direito na sucessora. Ademais, eventual diferença a menor em relação ao saldo na data da aquisição societária(registrado na parte B), poderá ser excluído do Lucro Real do período em que o bem ou direito for sendo realizado na sucessora(p.ex. depreciação). Contudo, se determinado bem ou direito que lhe deu causa não estiver presente no patrimônio da investida na data do evento societário, o correspondente valor de mais-valia deve ser baixado do saldo original registrado na Parte B, sem afetar o Lucro Real.(art. 100, inc.III, c, Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020112,1.02.02.01.12,"(-) Menos Valia em Investimentos - no País","Contas que registram a menos-valia, referente a aquisições de participações societárias no país, apurada segundo regramentos societários, e tratada no art. 20, inc.II, Decreto-Lei nº 1.598/1977. A amortização societária ocorrerá, conforme bem ou direito for realizado na investida, podendo valor ser excluído do Lucro Real do período(art. 25, Decreto-Lei nº 1.598/1977) e posteriormente ser necessário adicionar, quando da alienação da participação, para fins de cálculo do ganho ou perda de capital(art. 33, inc. II, Decreto-Lei nº 1.598/1977 ). O saldo original apurado na data da aquisição societária deve ser registrado na Parte B do eLalur, somente vindo a ser considerado integrante do custo do bem ou direito que lhe deu causa, ou ser adicionado ao Lucro Real inclusive em parcelas mensais, nos termos do art. 21 da Lei nº 12.973/2014. No caso do referido bem ou direito e a menos-valia correlata fundirem-se em um mesmo patrimônio, o saldo da menos-valia registrado na investidora na data do evento societário(p.ex. incorporação) integrará o custo do bem ou direito na sucessora. Ademais, eventual diferença a menor em relação ao saldo na data da aquisição societária(registrado na parte B), deverá ser adicionado ao Lucro Real do período em que o bem ou direito for sendo realizado na sucessora(p.ex. depreciação). Contudo, se determinado bem ou direito que lhe deu causa não estiver presente no patrimônio da investida na data do evento societário, o correspondente valor de mais-valia deve ser baixado do saldo original registrado na Parte B, sem afetar o Lucro Real(art. 101, §4º, da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020120,1.02.02.01.20,"Ágios em Investimentos Gerados até 31/12/2009 - no País","Contas que registram o ágio gerado até 13/12/2009, antes entrada em vigor CPC 15, em aquisições de participação societárias no país, fundamentado por diferença de valor de mercado dos bens, por valor de rentabilidade futura, por fundo de comércio, intangíveis, ou outras razões econômicas(art. 20-redação original, Decreto-Lei nº 1.598/1977). A amortização societária destes valores cessaria a partir do exercício social iniciado em 01/01/2009, nos termos do item 50, CPC13. Todavia, na vigência do RTT, poderiam ocorrer amortizações com efeito fiscal, registradas exclusivamente em FCONT, ainda sob efeito dos arts. 7º e 8º da Lei nº 9.532/1997. Os valores de ágio calculados por esta metodologia não mais vigente, contudo, decorrente da aplicação de regramento transitório previsto no art. 65, Lei nº 12.973/2014, devem ser controlados extracontabilmente em memória de cálculo encaminhada para ECF no registro Y800(art. 107, da Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102020121,1.02.02.01.21,"(-) Deságios em Investimentos Gerados até 31/12/2009 - no País","Contas que registram ao deságio gerado até 13/12/2009, antes entrada em vigor CPC 15, em aquisições de participação societárias no país, fundamentado por diferença de valor de mercado dos bens, por valor de rentabilidade futura, por fundo de comércio, intangíveis, ou outras razões econômicas(art. 20-redação original, Decreto-Lei nº 1.598/1977). A amortização societária destes valores cessaria a partir do exercício social iniciado em 01/01/2009, nos termos do item 50, CPC13. Todavia, na vigência do RTT, poderiam ocorrer amortizações com efeito fiscal, registradas exclusivamente em FCONT, ainda sob efeito dos arts. 7º e 8º da Lei nº 9.532/1997. Os valores de deságio calculados por esta metodologia não mais vigente, contudo, decorrente da aplicação de regramento transitório previsto no art. 65, Lei nº 12.973/2014, devem ser controlados extracontabilmente em memória de cálculo encaminhada para ECF no registro Y800 (art. 107, da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020140,1.02.02.01.40,"(-) Lucros a Apropriar em Vendas com Controladas -no País","Contas que registram lucros não realizados de vendas a empresas controladas no país, nos termos dos item 28 , CPC 18(downstream). Em caso de eventual ausência de registros contábeis, deve-se proceder o ajuste ao resultado do período por meio do eLalur, art. 61, Lei nº 12.973/2014.",asset_non_current,,l10n_br_account_chart_template
account_template_102020141,1.02.02.01.41,"(-) Lucros a Apropriar em Vendas com Coligadas -no País","Contas que registram lucros não realizados de vendas a empresas coligadas no país, nos termos dos item 28 , CPC 18(downstream). Em caso de eventual ausência de registros contábeis, deve-se proceder o ajuste ao resultado do período por meio do eLalur, art. 61, Lei nº 12.973/2014.",asset_non_current,,l10n_br_account_chart_template
account_template_102020142,1.02.02.01.42,"(-) Lucros a Apropriar em Vendas com Joint Ventures -no País","Contas que registram lucros não realizados de vendas a joint ventures, nos termos dos item 28 , CPC 18(downstream). Em caso de eventual ausência de registros contábeis, deve-se proceder o ajuste ao resultado do período por meio do eLalur, art. 61, Lei nº 12.973/2014.",asset_non_current,,l10n_br_account_chart_template
account_template_102020143,1.02.02.01.43,"(-) Lucros a Apropriar em Vendas com outras sociedades - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no País","Contas que registram lucros não realizados de vendas a outras empresas(downstream), nos termos dos item item 28 , CPC 18. Em caso de eventual ausência de registros contábeis, deve-se proceder o ajuste ao resultado do período por meio do eLalur, art. 61, Lei nº 12.973/2014.",asset_non_current,,l10n_br_account_chart_template
account_template_102020155,1.02.02.01.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) de Participações Permanentes Avaliadas pelo Método de Equivalência Patrimonial - no País","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuberabilidade do valor contábil das participações societárias no pais, avaliadas pela equivalência patrimonial. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do e-Lalur até ocorrência da alienação ou baixa da participação societária, quando poderão ser excluídos do Lucro Real (art. 25, Decreto-Lei nº 1.598/1977)",asset_non_current,,l10n_br_account_chart_template
account_template_102020156,1.02.02.01.56,"(-) Perdas por Redução ao Valor Recuperável (Impairment) do Goodwill em Investimentos - no País","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil do goodwill em participações societárias no país. Estes valores não estão sujeitos a reversão, conforme item 124, CPC 01. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do e-Lalur até ocorrência da alienação ou baixa da participação societária, quando poderão ser excluídos do Lucro Real (art. 25, Decreto-Lei nº 1.598/1977) . Esta conta não registra impairment sobre goodwill gerado em outras modalidades de combinações de negócio previsas no CPC 15, bem como, em eventos societários de incorporação, fusão e cisão, cujos valores devem ser registrados em conta de intangível(1.02.05.01.56).",asset_non_current,,l10n_br_account_chart_template
account_template_102020160,1.02.02.01.60,"Subconta - Ajuste a Valor Justo (AVJ) Reflexo - Ganho ou Perda na Investida","Contas que registram os ganhos ou perdas na mensuração de participação societária, no país, avaliada pelo método da equivalência patrimonial, decorrentes de alterações do valor justo de ativos ou passivos na investida. Se a investidora possuir saldos de mais-valia ou menos-valia derivados destes mesmos bens, deve primeiro baixar estes valores(art. 24-A/24-B, Decreto-Lei nº 1.598/1977). Somente se o AVJ for referente a outros bens da investida, ou já tenha ocorrido exaurimento dos correlatos mais ou menos-valia, então passará a registrar estes ganhos ou perdas reflexas em subconta na investidora. Caso bem que sofra o AVJ for único, a descrição da subconta deve identificá-lo, em sendo vários, pode utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §4º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102020165,1.02.02.01.65,"Subconta - Ajuste a Valor Justo (AVJ) de Subscrição de Capital - Ganho ou Perda de Capital","Contas que registram os ganhos ou perdas em bens do ativo, inclusive participações societárias, quando avaliados a valor justo, na transmissão para subscrição de capital social, ou de valores mobiliários em outra empresa. Estes valores devem ser excluídos ou adicionados, respectivamente, no lucro real do exercício em que ocorrer a subscrição, devendo ocorrer a posterior adição ou exclusão conforme hipóteses de realização disciplinadas nos arts.17/18, Lei nº 12.973/2014. Caso bem que sofra o AVJ for único, a descrição da subconta deve identificá-lo, em sendo vários, pode utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §4º, Instrução Normativa SRF nº 1.515/2014). Excepcionalmente, recebe esta classificação ainda que a investida ou emitente dos valores mobiliários tenha sede ou domicílio no exterior.",asset_non_current,,l10n_br_account_chart_template
account_template_102020175,1.02.02.01.75,"(-) Subconta - Ajuste a Valor Presente (AVP) de Participação Societária","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de participações societárias, no país, avaliadas pelo método da equivalência patrimonial. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até realização da participação, quando valor do AVP poderá ser excluído do Lucro Real do período(art.5º, Lei nº 12.973/2014 c/c art. 39 , Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020180,1.02.02.01.80,"Subconta - Mais Valia da Participação Anterior - Estágios","Contas que registram a mais-valia relativa à participação societária anterior, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa deste valor e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020181,1.02.02.01.81,"(-) Subconta - Menos Valia da Participação Anterior - Estágios","Contas que registram a menos-valia relativa à participação societária anterior, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa deste valor e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020182,1.02.02.01.82,"Subconta - Goodwill da Participação Anterior - Estágios","Contas que registram o goodwill relativo à participação societária anterior, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa deste valor e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020184,1.02.02.01.84,"Subconta - Variação de Mais Valia da Participação Anterior - Estágios","Contas que registram as alterações positivas ou negativas na mais-valia relativa à participação societária anterior, quando reavaliadas a valor justo, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa do valor anterior e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020185,1.02.02.01.85,"(-) Subconta - Variação de Menos Valia da Participação Anterior - Estágios","Contas que registram as alterações positivas ou negativas na menos-valia relativa à participação societária anterior, quando reavaliadas a valor justo, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa do valor anterior e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020186,1.02.02.01.86,"Subconta - Variação de Goodwill da Participação Anterior - Estágios","Contas que registram as alterações positivas ou negativas no goodwill relativo à participação societária anterior, quando reavaliadas a valor justo, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa do valor anterior e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020201,1.02.02.02.01,"Participações Permanentes em Controladas - no Exterior","Contas que registram investimentos permanentes no exterior, na forma de participação em outras sociedades nas quais se detenham o controle.",asset_non_current,,l10n_br_account_chart_template
account_template_102020202,1.02.02.02.02,"Subconta - Tributação em Base Universais (TBU) - Controladas Diretas - No Exterior","Contas que registram o resultado contábil na variação do valor do investimento equivalente aos lucros ou prejuízos auferidos pelas controladas diretas no exterior. A empresa controladora domiciliada no Brasil, mesmo que equiparada(art. 83, Lei nº 12.973/2014), deve registrar em subcontas individuais a parcela do ajuste no valor do investimento equivalente aos lucros auferidos por estas empresas antes da tributação no exterior sobre o lucro(art.76, Lei nº 12.973/2014, c/c arts. 2º a 4º da Instrução Normativa SRF nº 1.520/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102020203,1.02.02.02.03,"Subconta - Tributação em Base Universais (TBU) - Controladas Indiretas - no Exterior","Contas que registram o resultado contábil na variação do valor do investimento equivalente aos lucros ou prejuízos auferidos pelas controladas indiretas no exterior. A empresa controladora domiciliada no Brasil, mesmo que equiparada(art. 83, Lei nº 12.973/2014), deve registrar em subcontas individuais a parcela do ajuste no valor do investimento equivalente aos lucros auferidos por estas empresas antes da tributação no exterior sobre o lucro(art.76, Lei nº 12.973/2014, c/c arts. 2º a 4º da Instrução Normativa SRF nº 1.520/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102020204,1.02.02.02.04,"Participações Permanentes em Coligadas - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior","Contas que registram investimentos permanentes no exterior, na forma de participação em outras sociedades nas quais se tenha influência significativa.",asset_non_current,,l10n_br_account_chart_template
account_template_102020205,1.02.02.02.05,"Participações Permanentes em Joint Ventures - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior","Contas que registram investimentos permanentes no exterior, na forma de participação em joint ventures, avaliados pelo Método de Equivalência Patrimonial.",asset_non_current,,l10n_br_account_chart_template
account_template_102020206,1.02.02.02.06,"Participações Permanentes em Outras Sociedades do Mesmo Grupo ou Controle Comum - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior","Contas que registram investimentos permanentes no exterior, na forma de participação permanente em outras sociedades do grupo ou controle comum, sem que haja relação de controle ou influência significativa, avaliados pelo Método de Equivalência Patrimonial(art. 248, Lei nº 6.404/1976).",asset_non_current,,l10n_br_account_chart_template
account_template_102020210,1.02.02.02.10,"Goodwill em Investimentos - no Exterior","Contas que registram o goodwill, gerado em participações societária no exterior.",asset_non_current,,l10n_br_account_chart_template
account_template_102020211,1.02.02.02.11,"Mais Valia em Investimentos - no Exterior","Contas que registram mais-valia referente a investimentos no exterior.",asset_non_current,,l10n_br_account_chart_template
account_template_102020212,1.02.02.02.12,"(-) Menos Valia em Investimentos - no Exterior","Contas que registram menos-valia referente a investimentos no exterior.",asset_non_current,,l10n_br_account_chart_template
account_template_102020220,1.02.02.02.20,"Ágios em Investimentos Gerados até 31/12/2009 - no Exterior","Contas que registram ao ágio gerado até 13/12/2009, antes entrada em vigor CPC 15, por diferença de valor de mercado dos bens, por valor de rentabilidade futura, por fundo de comércio, intangíveis, ou outras razões econômicas, gerados em aquisições societários no exterior.",asset_non_current,,l10n_br_account_chart_template
account_template_102020221,1.02.02.02.21,"(-) Deságios em Investimentos Gerados até 31/12/2009 - no Exterior","Contas que registram ao deságio gerado até 13/12/2009, antes entrada em vigor CPC 15, por diferença de valor de mercado dos bens, por valor de rentabilidade futura, por fundo de comércio, intangíveis, ou outras razões econômicas, gerados em aquisições societários no exterior.",asset_non_current,,l10n_br_account_chart_template
account_template_102020240,1.02.02.02.40,"(-) Lucros a Apropriar em Vendas com Controladas - no Exterior","Contas que registram lucros não realizados de vendas a empresas controladas no exterior, nos termos dos item 28 , CPC 18(downstream). Em caso de eventual ausência de registros contábeis, deve-se proceder o ajuste ao resultado do período por meio do eLalur( art. 61, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020241,1.02.02.02.41,"(-) Lucros a Apropriar em Vendas com Coligadas - no Exterior","Contas que registram lucros não realizados de vendas a empresas coligadas no exterior, nos termos dos item 28 , CPC 18(downstream). Em caso de eventual ausência de registros contábeis, deve-se proceder o ajuste ao resultado do período por meio do eLalur( art. 61, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020242,1.02.02.02.42,"(-) Lucros a Apropriar em Vendas com Joint Ventures - no Exterior","Contas que registram lucros não realizados de vendas a joint ventures no exterior, nos termos dos item 28 , CPC 18(downstream). Em caso de eventual ausência de registros contábeis, deve-se proceder o ajuste ao resultado do período por meio do eLalur(art. 61, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020243,1.02.02.02.43,"(-) Lucros a Apropriar em Vendas com outras sociedades - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior","Contas que registram lucros não realizados de vendas a outras empresas(downstream), nos termos dos item item 28 , CPC 18. Em caso de eventual ausência de registros contábeis, deve-se proceder o ajuste ao resultado do período por meio do eLalur(art. 61, Lei nº 12.973/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102020255,1.02.02.02.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) em Participações Permanentes Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil dos investimentos em participações societárias no exterior. Referidos valores devem ser adicionadas ao Lucro Real do período(art. 95, §2º, Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020256,1.02.02.02.56,"(-) Perdas por Redução ao Valor Recuperável (Impairment) do Goodwill em Investimentos - no Exterior","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuberabilidade do valor contábil do goodwill gerado em participações societária no exterior. Estes valores não estão sujeitos a reversão, conforme item 124, CPC 01, devendo ser adicionadas ao Lucro Real do período(art. 95, §2º, Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020260,1.02.02.02.60,"Subconta - Ajuste a Valor Justo (AVJ) Reflexo - Ganho ou Perda na Investida – no Exterior","Contas que registram os ganhos ou perdas na mensuração de participação societária, no exterior, avaliada pelo método da equivalência patrimonial, decorrentes de alterações do valor justo de ativos ou passivos na investida. Se a investidora possuir saldos de mais-valia ou menos-valia derivados destes mesmos bens, deve primeiro baixar estes valores(art. 24-A/24-B, Decreto-Lei nº 1.598/1977). Somente se o AVJ for referente a outros bens da investida, ou já tenha ocorrido exaurimento dos correlatos mais ou menos-valia, então passará a registrar estes ganhos ou perdas reflexas em subconta na investidora. Caso bem que sofra o AVJ for único, a descrição da subconta deve identificá-lo, em sendo vários, pode utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §4º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102020265,1.02.02.02.65,"Subconta - Ajuste a Valor Justo (AVJ) de Subscrição de Capital - Ganho ou Perda de Capital – no Exterior","Contas que registram os ganhos ou perdas em bens do ativo, inclusive participações societárias, quando avaliados a valor justo, na transmissão para subscrição de capital social, ou de valores mobiliários em outra empresa. Estes valores devem ser excluídos ou adicionados, respectivamente, no lucro real do exercício em que ocorrer a subscrição, devendo ocorrer a posterior adição ou exclusão conforme hipóteses de realização disciplinadas nos arts.17/18, Lei nº 12.973/2014. Caso bem que sofra o AVJ for único, a descrição da subconta deve identificá-lo, em sendo vários, pode utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §4º, Instrução Normativa SRF nº 1.515/2014). Excepcionalmente, recebe esta classificação ainda que a investida ou emitente dos valores mobiliários tenha sede ou domicílio no exterior.",asset_non_current,,l10n_br_account_chart_template
account_template_102020275,1.02.02.02.75,"(-) Subconta - Ajuste a Valor Presente (AVP) de Participação Societária – no Exterior","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de participações societárias, no país, avaliadas pelo método da equivalência patrimonial. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até realização da participação, quando valor do AVP poderá ser excluído do Lucro Real do período(art.5º, Lei nº 12.973/2014 c/c art. 39 , Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020280,1.02.02.02.80,"Subconta - Mais Valia da Participação Anterior - Estágios – no Exterior","Contas que registram a mais-valia relativa à participação societária anterior, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa deste valor e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020281,1.02.02.02.81,"(-) Subconta - Menos Valia da Participação Anterior - Estágios – no Exterior","Contas que registram a menos-valia relativa à participação societária anterior, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa deste valor e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020282,1.02.02.02.82,"Subconta - Goodwill da Participação Anterior - Estágios – no Exterior","Contas que registram o goodwill relativo à participação societária anterior, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa deste valor e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020284,1.02.02.02.84,"Subconta - Variação de Mais Valia da Participação Anterior - Estágios – no Exterior","Contas que registram as alterações positivas ou negativas na mais-valia relativa à participação societária anterior, quando reavaliadas a valor justo, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa do valor anterior e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020285,1.02.02.02.85,"(-) Subconta - Variação de Menos Valia da Participação Anterior - Estágios – no Exterior","Contas que registram as alterações positivas ou negativas na menos-valia relativa à participação societária anterior, quando reavaliadas a valor justo, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa do valor anterior e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020286,1.02.02.02.86,"Subconta - Variação de Goodwill da Participação Anterior - Estágios – no Exterior","Contas que registram as alterações positivas ou negativas no goodwill relativo à participação societária anterior, quando reavaliadas a valor justo, nos casos de aquisição de participações em estágios. Ainda que a norma contábil possa determinar a baixa do valor anterior e reconhecimento de nova mensuração das parcelas envolvidas na participação, este valor deve ficar apartado em subconta de controle, submetendo-se ao tratamento fiscal adequado, conforme arts. 37/39, Lei nº 12.973/2014. A operacionalização desta subconta é melhor evidenciada no exemplo constante no anexoII da Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020301,1.02.02.03.01,"Imóveis Próprios em Construção - Propriedades para Investimento","Contas que registram os imóveis próprios em construção, destinados a auferir aluguel ou para valorização do capital, ou para ambas, desde que não utilizados na exploração ou na manutenção das atividades da empresa ou se destinem à venda no curso ordinário do negócio. Neste último caso, constarão em estoques. Consoante disposto nos itens 44/48, ICPC10, devem receber esta classificação ainda que o objeto da pessoa jurídica seja locação de imóveis, deixando para o imobilizado apenas quando aluguel estiver vinculado a ativo complementar na produção ou no fornecimento de bens ou serviços. Nos termos do CPC 28, sua mensuração inicial deve ser pelo custo, já nas subsequentes, a depender de sua política contábil, regra geral, escolher entre o método do valor justo ou do custo. Seu item 53 contempla tratamento específico para propriedade em construção.",asset_non_current,,l10n_br_account_chart_template
account_template_102020305,1.02.02.03.05,"Imóveis Próprios – Pelo Custo - Propriedades para Investimento","Contas que registram os imóveis próprios mensurados pelo custo, destinados a auferir aluguel ou para valorização do capital, ou para ambas, desde que não utilizados na exploração ou na manutenção das atividades da empresa ou se destinem à venda no curso ordinário do negócio. Neste último caso, constarão em estoques. Consoante disposto nos itens 44/48, ICPC10, devem receber esta classificação ainda que o objeto da pessoa jurídica seja locação de imóveis, deixando para o imobilizado apenas quando aluguel estiver vinculado a ativo complementar na produção ou no fornecimento de bens ou serviços. Nos termos do CPC 28, sua mensuração inicial deve ser pelo custo, exceto custo atribuível resultante de nova mensuração, ocorrida nos termos dos itens 20/27, ICPC 10. Já nas subsequentes, a depender de sua política contábil, regra geral, escolher entre o método do valor justo ou do custo, sendo uniforme para todas propriedades para investimento. Ainda que utilize o custo para mensuração, deve divulgar o valor justo em Nota Explicativa encaminhada pela ECD.",asset_non_current,,l10n_br_account_chart_template
account_template_102020306,1.02.02.03.06,"Imóveis Próprios – Valor Justo - Propriedades para Investimento","Contas que registram os imóveis próprios mensurados a valor justo, destinados a auferir aluguel ou para valorização do capital, ou para ambas, desde que não utilizados na exploração ou na manutenção das atividades da empresa ou se destinem à venda no curso ordinário do negócio. Neste último caso, constarão em estoques. Consoante disposto nos itens 44/48, ICPC10, devem receber esta classificação ainda que o objeto da pessoa jurídica seja locação de imóveis, deixando para o imobilizado apenas quando aluguel estiver vinculado a ativo complementar na produção ou no fornecimento de bens ou serviços.Nos termos do CPC 28, sua mensuração inicial deve ser pelo custo, já nas subsequentes, a depender de sua política contábil, regra geral, escolher entre o método do valor justo ou do custo, sendo uniforme para todas propriedades para investimento. Uma vez adotado valor justo, deve manter o método.",asset_non_current,,l10n_br_account_chart_template
account_template_102020308,1.02.02.03.08,"Imóveis Objeto de Arrendamento – Propriedades para Investimento","Contas que registram os imóveis da arrendatária, destinados a auferir aluguel ou para valorização do capital, ou para ambas, desde que não utilizados na exploração ou na manutenção das atividades da empresa ou se destinem à venda no curso ordinário do negócio. Consoante disposto nos itens 44/48, ICPC10, devem receber esta classificação ainda que o objeto da pessoa jurídica seja locação de imóveis, deixando para o imobilizado apenas quando aluguel estiver vinculado a ativo complementar na produção ou no fornecimento de bens ou serviços.Nos termos do CPC 28, item 25, sua mensuração inicial deve ser o menor entre o valor justo da propriedade e o valor presente dos pagamentos mínimos do arrendamento. Nas mensurações subsequentes, a depender de sua política contábil, regra geral, escolher entre o método do valor justo ou do custo, salvo se receber o imóvel em arrendamento operacional, quando método pelo justo será obrigatório, item 34. A metodologia deve ser uniforme para todas propriedades para investimento. Uma vez adotado valor justo, deve manter o método.",asset_non_current,,l10n_br_account_chart_template
account_template_102020330,1.02.02.03.30,"(-) Depreciação Acumuladas – Propriedades para Investimento","Contas que registram as depreciações acumuladas das propriedades para investimento mensuradas pelo custo.",asset_non_current,,l10n_br_account_chart_template
account_template_102020355,1.02.02.03.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Propriedades para Investimento","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuberabilidade do valor contábil dos ativos classificados como propriedade para investimento. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020370,1.02.02.03.70,"Subconta - Ajuste a Valor Justo – Propriedades para Investimentos.","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre propriedades para investimento, inclusive decorrentes apenas de sua mensuração inicial, sendo neste caso reconhecida a diferença com o valor contábil no resultado, salvo quando aumento superior a eventual perda anterior por impairmente, quando deverá ser creditado diretamente no Patrimônio Líquido, em ajustes de avaliação patrimonial(CPC 28, item 62). Referidos valores deverão ser computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102020371,1.02.02.03.71,"Subconta - Ajuste Valor Justo – Depreciação Acumulada – Propriedade para Investimento","Contas que registram a depreciação acumulada sobre o ajuste a valor justo registrado na subconta 1.02.02.03.70. A operacionalização desta subconta está evidenciada nos exemplos 5/6, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102020375,1.02.02.03.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Propriedades para Investimento","Contas que registram os ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de ativos não circulantes mantidos para venda ou da correlata despesa de vendas. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até que bem seja realizado, quando valor do AVP poderá ser excluído do Lucro Real do período(art.5º, Lei 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102020376,1.02.02.03.76,"Subconta - Ajuste Valor Presente – Depreciação Acumulada - Propriedades para Investimento","Contas que registram a depreciação acumulada sobre o ajuste a valor presente registrado na subconta 1.02.02.03.75. A operacionalização desta subconta está evidenciada no exemplo 4, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102020390,1.02.02.03.90,"Subconta – Adoção Inicial – Propriedades para Investimento","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014), inclusive decorrentes do custo atribuível resultante de nova mensuração, ocorrida nos termos dos itens 20/27, ICPC 10. Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102020391,1.02.02.03.91,"Subconta – Adoção Inicial – Depreciação Acumulada - Propriedades para Investimento","Contas que registram a depreciação acumulada sobre o ajuste adoção inicial registrado na subconta 1.02.02.03.90. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102020395,1.02.02.03.95,"Subconta – Adoção Inicial - Taxa de Depreciação Diferente - Propriedades para Investimento","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor da depreciação acumulada mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Esta conta registra apenas a diferença gerada, durante a vigência do RTT, pelo uso de taxas de depreciação diferentes das definidas nos anexos I e II da Instrução Normativa SRF nº 162/1998. Detalhes sobre a contabilização estão descritos no Anexo IV, Instrução Normativa SRF nº 1.515/2014). Após a adoção da Lei nº 12.973/2014 estas diferenças posteriormente geradas passarão a ser controladas exclusivamente no eLalur(art. 168, Instrução Normativa SRF nº 1.515/2014). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102021003,1.02.02.10.03,"Investimentos Decorrentes de Incentivos Fiscais","Contas que registram os investimentos decorrentes de incentivos fiscais representados por ações novas da Embraer ou de empresas nacionais de informática ou por participação direta decorrente da troca do CI – Certificado de Investimento por ações pertencentes às carteiras de Fundos (Finor, Finam e Fites). Inclui-se a aquisição de quotas representativas de direitos de comercialização sobre produção de obras audiovisuais cinematográficas brasileiras de produção independente, com projetos previamente aprovados pelo Ministério da Cultura, realizada no mercado de capitais, em ativos previstos em lei e autorizados pela Comissão de Valores Mobiliários (CVM).",asset_non_current,,l10n_br_account_chart_template
account_template_102021010,1.02.02.10.10,"Outros Investimentos Permanentes","Contas que registram outros investimentos não classificáveis em contas mais específicas.",asset_non_current,,l10n_br_account_chart_template
account_template_102021020,1.02.02.10.20,"(-) Outras Contas Retificadoras ­– Outros Investimentos Permanentes","Contas que registram outras contas retificadoras do grupo investimentos, não classificáveis em contas mais específicas.",asset_non_current,,l10n_br_account_chart_template
account_template_102021050,1.02.02.10.50,"( - ) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Outros Investimentos Permanentes","Contas que registram os ajustes a valor presente efetuados sobre os outros investimentos permanentes. Referidos valores serão apropriados ao resultado pelo regime de competência e excluídos do Lucro Real. Não devem ser controlados em subcontas, mas na Parte B do eLalur, sendo seu montante adicionado ao Lucro Real no mesmo período em a receita ou resultado da operação deva ser tributada(art.4º, Lei 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102021055,1.02.02.10.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Outros Investimentos","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuberabilidade do valor contábil dos ativos classificados como outros investimento. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102021070,1.02.02.10.70,"Subconta - Ajuste a Valor Justo - Outros Investimentos Permanentes","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os outros investimentos permanentes, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da alienação ou baixa do ativo(arts. 49/53, Instrução Normativa SRF Nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102021090,1.02.02.10.90,"Subconta – Adoção Inicial - Outros Investimentos Permanentes","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD (art. 169, §6º, Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102030101,1.02.03.01.01,"Terrenos","Contas que registram os terrenos de propriedade da pessoa jurídica utilizados nas operações, ou seja, onde se localizam a fábrica, os depósitos, os escritórios, as filiais, as lojas, etc.
Atenção: O valor do terreno onde está em construção uma nova unidade que ainda não esteja em operação também deve ser informada nesta conta.",asset_fixed,,l10n_br_account_chart_template
account_template_102030102,1.02.03.01.02,"Edifícios e Construções","Contas que registram os edifícios, melhoramentos e obras integradas aos terrenos, e os serviços e instalações provisórias, necessários à construção e ao andamento das obras, tais como: limpeza do terreno, serviços topográficos, sondagens de reconhecimento, terraplenagem, e outras similares. Atenção: As construções em andamento devem ser informadas na conta Construções em Andamento.",asset_fixed,,l10n_br_account_chart_template
account_template_102030103,1.02.03.01.03,"Construções em Andamento - Imóvel Próprio","Contas que registram as construções em andamento de edifícios, melhoramentos e obras integradas aos terrenos, e os serviços e instalações provisórias, necessários à construção e ao andamento das obras em imóvel próprio da entidade, tais como: limpeza do terreno, serviços topográficos, sondagens de reconhecimento, terraplenagem, e outras similares.",asset_fixed,,l10n_br_account_chart_template
account_template_102030104,1.02.03.01.04,"Outras Imobilizações em Andamento","Contas que registram as construções em andamento de edifícios, melhoramentos e obras integradas aos terrenos, e os serviços e instalações provisórias, necessários à construção e ao andamento das obras, tais como: limpeza do terreno, serviços topográficos, sondagens de reconhecimento, terraplenagem, e outras similares, inclusive os decorrentes de operações que transfiram à companhia os benefícios, riscos e controle desses bens",asset_fixed,,l10n_br_account_chart_template
account_template_102030105,1.02.03.01.05,"Benfeitorias em Imóveis de Terceiros","Contas que registram as construções, instalações e outras benfeitorias em terrenos, prédios ou edifícios alugados de uso administrativo ou de produção.",asset_fixed,,l10n_br_account_chart_template
account_template_102030106,1.02.03.01.06,"Máquinas, Equipamentos e Instalações Industriais","Contas que registram os equipamentos, máquinas e instalações industriais utilizados no processo de produção da pessoa jurídica.",asset_fixed,,l10n_br_account_chart_template
account_template_102030107,1.02.03.01.07,"Móveis, Utensílios e Instalações Comerciais","Contas que registram os móveis, utensílios e instalações utilizados nas atividades administrativa e comercial da pessoa jurídica.",asset_fixed,,l10n_br_account_chart_template
account_template_102030108,1.02.03.01.08,"Veículos","Contas que registram os veículos de propriedade da pessoa jurídica. Atenção: Os veículos de uso direto na produção, como empilhadeiras e similares, devem ser informados na conta Equipamentos, Máquinas e Instalações Industriais.",asset_fixed,,l10n_br_account_chart_template
account_template_102030109,1.02.03.01.09,"Embarcações","Contas que registram as embarcações de propriedade da pessoa jurídica, utilizados nas atividades administrativas, comerciais ou produtivas.",asset_fixed,,l10n_br_account_chart_template
account_template_102030110,1.02.03.01.10,"Aeronaves","Contas que registram as aeronaves de propriedade da pessoa jurídica, utilizados nas atividades administrativas, comerciais ou produtivas.",asset_fixed,,l10n_br_account_chart_template
account_template_102030111,1.02.03.01.11,"Recursos Minerais","Contas que registram os direitos de exploração de jazidas de minério, de pedras preciosas, e similares.",asset_fixed,,l10n_br_account_chart_template
account_template_102030112,1.02.03.01.12,"Dutos e Tubulações","Contas que registram os dutos e tubulações de propriedade da pessoa jurídica.",asset_fixed,,l10n_br_account_chart_template
account_template_102030113,1.02.03.01.13,"Linhas de Transmissão Elétrica","Contas que registram as linhas de transmissão elétrica de propriedade da pessoa jurídica.",asset_fixed,,l10n_br_account_chart_template
account_template_102030114,1.02.03.01.14,"Antenas e Torres de Transmissão","Contas que registram as antenas e torres de transmissão de propriedade da pessoa jurídica do setor de telecomunicações.",asset_fixed,,l10n_br_account_chart_template
account_template_102030115,1.02.03.01.15,"Máquinas Empregadas na Atividade Rural","Contas que registram as máquinas empregadas na atividade rural.",asset_fixed,,l10n_br_account_chart_template
account_template_102030116,1.02.03.01.16,"Tratores e Demais Veículos Empregados na Atividade Rural","Contas que registram os tratores e demais veículos empregados na atividade rural.",asset_fixed,,l10n_br_account_chart_template
account_template_102030128,1.02.03.01.28,"Outras Imobilizações por Aquisição","Contas que registram outras imobilizações não classificadas em contas mais específicas.",asset_fixed,,l10n_br_account_chart_template
account_template_102030130,1.02.03.01.30,"(-) Depreciação Acumulada - Imobilizado","Contas que registram a depreciação acumulada das contas do imobilizado.",asset_fixed,,l10n_br_account_chart_template
account_template_102030131,1.02.03.01.31,"(-) Amortização Acumulada - Imobilizado","Contas que registram a amortização acumulada das contas do imobilizado.",asset_fixed,,l10n_br_account_chart_template
account_template_102030132,1.02.03.01.32,"(-) Exaustão Acumulada - Imobilizado","Contas que registram a exaustão acumulada das contas do imobilizado.",asset_fixed,,l10n_br_account_chart_template
account_template_102030155,1.02.03.01.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Imobilizado","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil do imobilizado. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei 12.973/2014).",asset_fixed,,l10n_br_account_chart_template
account_template_102030175,1.02.03.01.75,"(-) Subconta - Ajuste Valor Presente – Imobilizado","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de ativos imobilizados Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até que bem seja realizado, quando valor do AVP poderá ser excluído do Lucro Real do período(art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_fixed,,l10n_br_account_chart_template
account_template_102030176,1.02.03.01.76,"Subconta - Ajuste Valor Presente – Depreciação Acumulada - Imobilizado","Contas que registram a depreciação acumulada sobre o ajuste a valor presente registrado na subconta 1.02.03.01.75. A operacionalização desta subconta está evidenciada no exemplo 4, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_fixed,,l10n_br_account_chart_template
account_template_102030177,1.02.03.01.77,"Subconta - Ajuste Valor Presente – Amortização Acumulada - Imobilizado","Contas que registram a amortização acumulada sobre o ajuste a valor presente registrado na subconta 1.02.03.01.75. A operacionalização desta subconta está evidenciada no exemplo 4, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_fixed,,l10n_br_account_chart_template
account_template_102030178,1.02.03.01.78,"Subconta - Ajuste Valor Presente – Exaustão Acumulada - Imobilizado","Contas que registram a exaustão acumulada sobre o ajuste a valor presente registrado na subconta 1.02.03.01.75. A operacionalização desta subconta está evidenciada no exemplo 4, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_fixed,,l10n_br_account_chart_template
account_template_102030190,1.02.03.01.90,"Subconta – Adoção Inicial - Imobilizado","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014), inclusive decorrentes do custo atribuível resultante de nova mensuração, ocorrida nos termos dos itens 20/27, ICPC 10. Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF Nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_fixed,,l10n_br_account_chart_template
account_template_102030191,1.02.03.01.91,"Subconta – Adoção Inicial – Depreciação Acumulada - Imobilizado","Contas que registram a depreciação acumulada sobre o ajuste adoção inicial registrado na subconta 1.02.03.01.90. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_fixed,,l10n_br_account_chart_template
account_template_102030192,1.02.03.01.92,"Subconta – Adoção Inicial – Amortização Acumulada - Imobilizado","Contas que registram a amortização acumulada sobre o ajuste adoção inicial registrado na subconta 1.02.03.01.90. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_fixed,,l10n_br_account_chart_template
account_template_102030193,1.02.03.01.93,"Subconta – Adoção Inicial – Exaustão Acumulada - Imobilizado","Contas que registram a exaustão acumulada sobre o ajuste adoção inicial registrado na subconta 1.02.03.01.90. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_fixed,,l10n_br_account_chart_template
account_template_102030195,1.02.03.01.95,"Subconta – Adoção Inicial – Taxa Depreciação Diferente - Imobilizado","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor da depreciação acumulada mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Esta conta registra apenas a diferença gerada, durante a vigência do RTT, pelo uso de taxas de depreciação diferentes das definidas nos anexos I e II da Instrução Normativa SRF nº 162/1998. Detalhes sobre a contabilização estão descritos no Anexo IV, Instrução Normativa SRF nº 1.515/2014). Após a adoção da Lei nº 12.973/2014 estas diferenças posteriormente geradas passarão a ser controladas exclusivamente no eLalur (art. 168, Instrução Normativa SRF nº 1.515/2014). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_fixed,,l10n_br_account_chart_template
account_template_102030201,1.02.03.02.01,"Veículos","Contas que registram os veículos recebidos em arrendamento, ou seja, no qual são substancialmente transferidos à arrendatária os riscos e benefícios inerentes à propriedade. O contrato é, ou contém, um arrendamento se ele transmite o direito de controlar o uso de ativo identificado por um período de tempo em troca de contraprestação, conforme item 9 do CPC06 (R2), sendo que o título de propriedade pode ou não vir a ser transferido. A classificação depende da essência da transação e não da forma do contrato. Nos termos do item 23 do CPC06 (R2), no reconhecimento inicial estes ativos devem ser mensurados ao custo, e de acordo com o item 24 do mesmo pronunciamento contábil, o custo do ativo de direito de uso deve compreender: a) o valor da mensuração inicial do passivo de arrendamento; b) quaisquer pagamentos de arrendamento já efetuados, menos quaisquer incentivos de arrendamento recebidos; c) quaisquer custos diretos iniciais do arrendatário; e d) estimativa de custos a serem incorridos na desmontagem e remoção do ativo subjacente. Para fins fiscais, a pessoa jurídica arrendatária deverá adicionar ao Lucro Real qualquer despesa com depreciação, amortização ou exaustão (art.13, Decreto-Lei nº 1.598/1977), todavia, poderá excluir as contraprestações pagas ou creditadas (art. 47, Lei nº 12.973/2014). Os valores decorrentes do Ajuste a Valor Presente efetivado sobre a dívida do contrato também devem ser adicionados ao Lucro Real, conforme venham a ser reconhecidos como despesa financeira (art. 48, Lei nº 12.973/2014). Maiores detalhes no art. 175 da Instrução Normativa RFB nº 1.700/2017. As diferenças percebidas nos valores do ativo, tratadas na adoção inicial da Lei nº 12.973/2014¸ arts. 66 e 67, não devem gerar controle em subcontas, conforme art. 303, Instrução Normativa RFB nº 1.700/2017.",asset_fixed,,l10n_br_account_chart_template
account_template_102030202,1.02.03.02.02,"Embarcações","Contas que registram as embarcações recebidas em arrendamento, ou seja, no qual são substancialmente transferidos à arrendatária os riscos e benefícios inerentes à propriedade. O contrato é, ou contém, um arrendamento se ele transmite o direito de controlar o uso de ativo identificado por um período de tempo em troca de contraprestação, conforme item 9 do CPC06 (R2), sendo que o título de propriedade pode ou não vir a ser transferido. A classificação depende da essência da transação e não da forma do contrato. Nos termos do item 23 do CPC06 (R2), no reconhecimento inicial estes ativos devem ser mensurados ao custo, e de acordo com o item 24 do mesmo pronunciamento contábil, o custo do ativo de direito de uso deve compreender: a) o valor da mensuração inicial do passivo de arrendamento; b) quaisquer pagamentos de arrendamento já efetuados, menos quaisquer incentivos de arrendamento recebidos; c) quaisquer custos diretos iniciais do arrendatário; e d) estimativa de custos a serem incorridos na desmontagem e remoção do ativo subjacente. Para fins fiscais, a pessoa jurídica arrendatária deverá adicionar ao Lucro Real qualquer despesa com depreciação, amortização ou exaustão (art.13, Decreto-Lei nº 1.598/1977), todavia, poderá excluir as contraprestações pagas ou creditadas (art. 47, Lei nº 12.973/2014). Os valores decorrentes do Ajuste a Valor Presente efetivado sobre a dívida do contrato também devem ser adicionados ao Lucro Real, conforme venham a ser reconhecidos como despesa financeira (art. 48, Lei nº 12.973/2014). Maiores detalhes no art. 175 da Instrução Normativa RFB nº 1.700/2017. As diferenças percebidas nos valores do ativo, tratadas na adoção inicial da Lei nº 12.973/2014¸ arts. 66 e 67, não devem gerar controle em subcontas, conforme art. 303, Instrução Normativa RFB nº 1.700/2017.",asset_fixed,,l10n_br_account_chart_template
account_template_102030203,1.02.03.02.03,"Aeronaves","Contas que registram as aeronaves recebidas em arrendamento, ou seja, no qual são substancialmente transferidos à arrendatária os riscos e benefícios inerentes à propriedade. O contrato é, ou contém, um arrendamento se ele transmite o direito de controlar o uso de ativo identificado por um período de tempo em troca de contraprestação, conforme item 9 do CPC06 (R2), sendo que o título de propriedade pode ou não vir a ser transferido. A classificação depende da essência da transação e não da forma do contrato. Nos termos do item 23 do CPC06 (R2), no reconhecimento inicial estes ativos devem ser mensurados ao custo, e de acordo com o item 24 do mesmo pronunciamento contábil, o custo do ativo de direito de uso deve compreender: a) o valor da mensuração inicial do passivo de arrendamento; b) quaisquer pagamentos de arrendamento já efetuados, menos quaisquer incentivos de arrendamento recebidos; c) quaisquer custos diretos iniciais do arrendatário; e d) estimativa de custos a serem incorridos na desmontagem e remoção do ativo subjacente. Para fins fiscais, a pessoa jurídica arrendatária deverá adicionar ao Lucro Real qualquer despesa com depreciação, amortização ou exaustão (art.13, Decreto-Lei nº 1.598/1977), todavia, poderá excluir as contraprestações pagas ou creditadas (art. 47, Lei nº 12.973/2014). Os valores decorrentes do Ajuste a Valor Presente efetivado sobre a dívida do contrato também devem ser adicionados ao Lucro Real, conforme venham a ser reconhecidos como despesa financeira (art. 48, Lei nº 12.973/2014). Maiores detalhes no art. 175 da Instrução Normativa RFB nº 1.700/2017. As diferenças percebidas nos valores do ativo, tratadas na adoção inicial da Lei nº 12.973/2014¸ arts. 66 e 67, não devem gerar controle em subcontas, conforme art. 303, Instrução Normativa RFB nº 1.700/2017.",asset_fixed,,l10n_br_account_chart_template
account_template_102030204,1.02.03.02.04,"Máquinas, Equipamentos e Instalações Industriais","Contas que registram máquinas, equipamentos e instalações industriais recebidos em arrendamento, ou seja, no qual são substancialmente transferidos à arrendatária os riscos e benefícios inerentes à propriedade. O contrato é, ou contém, um arrendamento se ele transmite o direito de controlar o uso de ativo identificado por um período de tempo em troca de contraprestação, conforme item 9 do CPC06 (R2), sendo que o título de propriedade pode ou não vir a ser transferido. A classificação depende da essência da transação e não da forma do contrato. Nos termos do item 23 do CPC06 (R2), no reconhecimento inicial estes ativos devem ser mensurados ao custo, e de acordo com o item 24 do mesmo pronunciamento contábil, o custo do ativo de direito de uso deve compreender: a) o valor da mensuração inicial do passivo de arrendamento; b) quaisquer pagamentos de arrendamento já efetuados, menos quaisquer incentivos de arrendamento recebidos; c) quaisquer custos diretos iniciais do arrendatário; e d) estimativa de custos a serem incorridos na desmontagem e remoção do ativo subjacente. Para fins fiscais, a pessoa jurídica arrendatária deverá adicionar ao Lucro Real qualquer despesa com depreciação, amortização ou exaustão (art.13, Decreto-Lei nº 1.598/1977), todavia, poderá excluir as contraprestações pagas ou creditadas (art. 47, Lei nº 12.973/2014). Os valores decorrentes do Ajuste a Valor Presente efetivado sobre a dívida do contrato também devem ser adicionados ao Lucro Real, conforme venham a ser reconhecidos como despesa financeira (art. 48, Lei nº 12.973/2014). Maiores detalhes no art. 175 da Instrução Normativa RFB nº 1.700/2017. As diferenças percebidas nos valores do ativo, tratadas na adoção inicial da Lei nº 12.973/2014¸ arts. 66 e 67, não devem gerar controle em subcontas, conforme art. 303, Instrução Normativa RFB nº 1.700/2017.",asset_fixed,,l10n_br_account_chart_template
account_template_102030205,1.02.03.02.05,"Móveis, Utensílios e Instalações Comerciais","Contas que registram móveis, utensílios e instalações comerciais recebidos em arrendamento, ou seja, no qual são substancialmente transferidos à arrendatária os riscos e benefícios inerentes à propriedade. O contrato é, ou contém, um arrendamento se ele transmite o direito de controlar o uso de ativo identificado por um período de tempo em troca de contraprestação, conforme item 9 do CPC06 (R2), sendo que o título de propriedade pode ou não vir a ser transferido. A classificação depende da essência da transação e não da forma do contrato. Nos termos do item 23 do CPC06 (R2), no reconhecimento inicial estes ativos devem ser mensurados ao custo, e de acordo com o item 24 do mesmo pronunciamento contábil, o custo do ativo de direito de uso deve compreender: a) o valor da mensuração inicial do passivo de arrendamento; b) quaisquer pagamentos de arrendamento já efetuados, menos quaisquer incentivos de arrendamento recebidos; c) quaisquer custos diretos iniciais do arrendatário; e d) estimativa de custos a serem incorridos na desmontagem e remoção do ativo subjacente. Para fins fiscais, a pessoa jurídica arrendatária deverá adicionar ao Lucro Real qualquer despesa com depreciação, amortização ou exaustão (art.13, Decreto-Lei nº 1.598/1977), todavia, poderá excluir as contraprestações pagas ou creditadas (art. 47, Lei nº 12.973/2014). Os valores decorrentes do Ajuste a Valor Presente efetivado sobre a dívida do contrato também devem ser adicionados ao Lucro Real, conforme venham a ser reconhecidos como despesa financeira (art. 48, Lei nº 12.973/2014). Maiores detalhes no art. 175 da Instrução Normativa RFB nº 1.700/2017. As diferenças percebidas nos valores do ativo, tratadas na adoção inicial da Lei nº 12.973/2014¸ arts. 66 e 67, não devem gerar controle em subcontas, conforme art. 303, Instrução Normativa RFB nº 1.700/2017.",asset_fixed,,l10n_br_account_chart_template
account_template_102030206,1.02.03.02.06,"Imóveis","Contas que registram os imóveis recebidos em arrendamento, ou seja, no qual são substancialmente transferidos à arrendatária os riscos e benefícios inerentes à propriedade. O contrato é, ou contém, um arrendamento se ele transmite o direito de controlar o uso de ativo identificado por um período de tempo em troca de contraprestação, conforme item 9 do CPC06 (R2), sendo que o título de propriedade pode ou não vir a ser transferido. A classificação depende da essência da transação e não da forma do contrato. Nos termos do item 23 do CPC06 (R2), no reconhecimento inicial estes ativos devem ser mensurados ao custo, e de acordo com o item 24 do mesmo pronunciamento contábil, o custo do ativo de direito de uso deve compreender: a) o valor da mensuração inicial do passivo de arrendamento; b) quaisquer pagamentos de arrendamento já efetuados, menos quaisquer incentivos de arrendamento recebidos; c) quaisquer custos diretos iniciais do arrendatário; e d) estimativa de custos a serem incorridos na desmontagem e remoção do ativo subjacente. Para fins fiscais, a pessoa jurídica arrendatária deverá adicionar ao Lucro Real qualquer despesa com depreciação, amortização ou exaustão (art.13, Decreto-Lei nº 1.598/1977), todavia, poderá excluir as contraprestações pagas ou creditadas (art. 47, Lei nº 12.973/2014). Os valores decorrentes do Ajuste a Valor Presente efetivado sobre a dívida do contrato também devem ser adicionados ao Lucro Real, conforme venham a ser reconhecidos como despesa financeira (art. 48, Lei nº 12.973/2014). Maiores detalhes no art. 175 da Instrução Normativa RFB nº 1.700/2017. As diferenças percebidas nos valores do ativo, tratadas na adoção inicial da Lei nº 12.973/2014¸ arts. 66 e 67, não devem gerar controle em subcontas, conforme art. 303, Instrução Normativa RFB nº 1.700/2017.",asset_fixed,,l10n_br_account_chart_template
account_template_102030209,1.02.03.02.09,"Outras Imobilizações por Arrendamento","Contas que registram outros bens registrados no grupo imobilizado recebidos em arrendamento, ou seja, no qual são substancialmente transferidos à arrendatária os riscos e benefícios inerentes à propriedade. O contrato é, ou contém, um arrendamento se ele transmite o direito de controlar o uso de ativo identificado por um período de tempo em troca de contraprestação, conforme item 9 do CPC06 (R2), sendo que o título de propriedade pode ou não vir a ser transferido. A classificação depende da essência da transação e não da forma do contrato. Nos termos do item 23 do CPC06 (R2), no reconhecimento inicial estes ativos devem ser mensurados ao custo, e de acordo com o item 24 do mesmo pronunciamento contábil, o custo do ativo de direito de uso deve compreender: a) o valor da mensuração inicial do passivo de arrendamento; b) quaisquer pagamentos de arrendamento já efetuados, menos quaisquer incentivos de arrendamento recebidos; c) quaisquer custos diretos iniciais do arrendatário; e d) estimativa de custos a serem incorridos na desmontagem e remoção do ativo subjacente. Para fins fiscais, a pessoa jurídica arrendatária deverá adicionar ao Lucro Real qualquer despesa com depreciação, amortização ou exaustão (art.13, Decreto-Lei nº 1.598/1977), todavia, poderá excluir as contraprestações pagas ou creditadas (art. 47, Lei nº 12.973/2014). Os valores decorrentes do Ajuste a Valor Presente efetivado sobre a dívida do contrato também devem ser adicionados ao Lucro Real, conforme venham a ser reconhecidos como despesa financeira (art. 48, Lei nº 12.973/2014). Maiores detalhes no art. 175 da Instrução Normativa RFB nº 1.700/2017. As diferenças percebidas nos valores do ativo, tratadas na adoção inicial da Lei nº 12.973/2014¸ arts. 66 e 67, não devem gerar controle em subcontas, conforme art. 303, Instrução Normativa RFB nº 1.700/2017.",asset_fixed,,l10n_br_account_chart_template
account_template_102030230,1.02.03.02.30,"(-) Depreciação Acumulada - Imobilizado - Bens objeto de arrendamento","Contas que registram a depreciação acumulada do imobilizado recebido em arrendamento.",asset_fixed,,l10n_br_account_chart_template
account_template_102030231,1.02.03.02.31,"(-) Amortização Acumulada - Imobilizado - Bens objeto de arrendamento","Contas que registram a depreciação acumulada do imobilizado recebido em arrendamento.",asset_fixed,,l10n_br_account_chart_template
account_template_102030232,1.02.03.02.32,"(-) Exaustão Acumulada - Imobilizado - Bens objeto de arrendamento","Contas que registram a depreciação acumulada do imobilizado recebido em arrendamento.",asset_fixed,,l10n_br_account_chart_template
account_template_102030255,1.02.03.02.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Imobilizado - Bens objeto de arrendamento","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil do imobilizado recebido em arrendamento. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_fixed,,l10n_br_account_chart_template
account_template_102030401,1.02.03.04.01,"Ativo Biológico de Produção - Origem Animal – Pelo Valor Justo","Contas que registram, nas empresas com atividade rural, os ativos biológicos para produção de origem animal, quando avaliados a valor justo. Nos termos do item 44 do CPC 29, seriam aqueles não consumíveis, autorrenováveis, como rebanhos de animais para produção de leite ou reprodução. Os produtos agrícolas devem receber classificação de estoques.",asset_non_current,,l10n_br_account_chart_template
account_template_102030402,1.02.03.04.02,"Ativo Biológico de Produção - Origem Vegetal – Pelo Valor Justo","Contas que registram, nas empresas com atividade rural, os ativos biológicos para produção de origem vegetal, quando avaliados a valor justo. Nos termos do item 44 do CPC 29, seriam aqueles não consumíveis, autorrenováveis, como árvores frutíferas, vinhas, árvores das quais se produz lenha por desbaste. Os produtos agrícolas devem receber classificação de estoques.",asset_non_current,,l10n_br_account_chart_template
account_template_102030403,1.02.03.04.03,"Ativo Biológico de Produção - Origem Animal – Pelo Custo","Contas que registram, nas empresas com atividade rural, os ativos biológicos para produção de origem animal, nos casos em que não seja possível mensurá-los ao valor justo, somente no reconhecimento inicial, devido a indisponibilidade de cotação de mercado e as alternativas não são, claramente, confiáveis. A pessoa jurídica que tenha mensurado previamente o ativo biológico ao seu valor justo, menos a despesa de venda, continuará a mensurá-lo assim até a sua venda. Nos termos do item 44 do CPC 29, seriam aqueles não consumíveis, autorrenováveis, como rebanhos de animais para produção de leite ou reprodução. Os produtos agrícolas devem receber classificação de estoques.",asset_non_current,,l10n_br_account_chart_template
account_template_102030404,1.02.03.04.04,"Ativo Biológico de Produção - Origem Vegetal - Pelo Custo","Contas que registram, nas empresas com atividade rural, os ativos biológicos para produção de origem vegetal, nos casos em que não seja possível mensurá-los ao valor justo, somente no reconhecimento inicial, devido a indisponibilidade de cotação de mercado e as alternativas não são, claramente, confiáveis. A pessoa jurídica que tenha mensurado previamente o ativo biológico ao seu valor justo, menos a despesa de venda, continuará a mensurá-lo assim até a sua venda. Nos termos do item 44 do CPC 29, seriam aqueles não consumíveis, autorrenováveis, como árvores frutíferas, vinhas, árvores das quais se produz lenha por desbaste. Os produtos agrícolas devem receber classificação de estoques.",asset_non_current,,l10n_br_account_chart_template
account_template_102030430,1.02.03.04.30,"(-) Depreciação Acumulada Ativos Biológicos de Produção","Contas que registram a depreciação acumulada de ativos biológicos de produção.",asset_non_current,,l10n_br_account_chart_template
account_template_102030432,1.02.03.04.32,"(-) Exaustão Acumulada Ativos Biológicos de Produção","Contas que registram a exaustão acumulada de ativos biológicos de produção.",asset_non_current,,l10n_br_account_chart_template
account_template_102030455,1.02.03.04.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Ativos Biológicos de Produção","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil do ativo biológico. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102030470,1.02.03.04.70,"Subconta - Ajuste a Valor Justo (AVJ) – Ativos Biológicos de Produção","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre ativos biológicos para produção. Referidos valores deverão ser registrados líquidos da despesa de venda e computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102030475,1.02.03.04.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Ativos Biológicos de Produção","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo dos ativos. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda ou baixa do ativo, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102030476,1.02.03.04.76,"Subconta - Ajuste Valor Presente – Depreciação Acumulada - Ativos Biológicos de Produção","Contas que registram a depreciação acumulada sobre o ajuste a valor presente registrado na subconta 1.02.03.04.75. A operacionalização desta subconta está evidenciada no exemplo 4, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102030478,1.02.03.04.78,"Subconta - Ajuste Valor Presente – Exaustão Acumulada - Ativos Biológicos de Produção","Contas que registram a exaustão acumulada sobre o ajuste a valor presente registrado na subconta 1.02.03.04.75. A operacionalização desta subconta está evidenciada no exemplo 4, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102030490,1.02.03.04.90,"Subconta – Adoção Inicial - Ativos Biológicos de Produção","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102030491,1.02.03.04.91,"Subconta – Adoção Inicial – Depreciação Acumulada – Ativo Biológico de Produção","Contas que registram a depreciação acumulada sobre o ajuste adoção inicial registrado na subconta 1.02.03.04.90. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102030493,1.02.03.04.93,"Subconta – Adoção Inicial – Exaustão Acumulada – Ativo Biológico de Produção","Contas que registram a exaustão acumulada sobre o ajuste adoção inicial registrado na subconta 1.02.03.04.90. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102030495,1.02.03.04.95,"Subconta – Adoção Inicial – Taxa Depreciação Diferente - Ativos Biológicos de Produção","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor da depreciação acumulada mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Esta conta registra apenas a diferença gerada, durante a vigência do RTT, pelo uso de taxas de depreciação diferentes das definidas nos anexos I e II da Instrução Normativa SRF nº 162/1998. Detalhes sobre a contabilização estão descritos no Anexo IV, Instrução Normativa SRF nº 1.515/2014). Após a adoção da Lei nº 12.973/2014 estas diferenças posteriormente geradas passarão a ser controladas exclusivamente no eLalur(art. 168, Instrução Normativa SRF nº 1.515/2014). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102030503,1.02.03.05.03,"Imobilizados Recebidos em Subvenções Governamentais","Contas que registram imobilizados adquiridos, construídos ou de outra forma recebidos em contrapartida a subvenções governamentais.",asset_non_current,,l10n_br_account_chart_template
account_template_102030504,1.02.03.05.04,"(-) Redutoras de Imobilizados Recebidos em Subvenções Governamentais","Contas redutoras de imobilizado recebidos em subvenções governamentais.",asset_non_current,,l10n_br_account_chart_template
account_template_102030528,1.02.03.05.28,"Outros Imobilizados","Contas que registram outras imobilizações da pessoa jurídica, não classificáveis em outras contas.",asset_non_current,,l10n_br_account_chart_template
account_template_102030529,1.02.03.05.29,"(-) Outras Contas Redutoras do Imobilizado","Outras contas redutoras do Imobilizado, inclusive a perda por redução do valor recuperável.",asset_non_current,,l10n_br_account_chart_template
account_template_102030530,1.02.03.05.30,"(-) Outras Depreciações, Amortizações e Quotas de Exaustão Acumuladas","Contas que registram as depreciações, amortizações e quotas de exaustão das contas de outros imobilizados.",asset_non_current,,l10n_br_account_chart_template
account_template_102030570,1.02.03.05.70,"Subconta - Ajuste a Valor Justo – Outros Imobilizados","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre outros imobilizados. Referidos valores deverão ser registrados líquidos da despesa de venda e computados na apuração Lucro Real à medida que ativo for realizado(arts. 13/14, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplos 5/6, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102030575,1.02.03.05.75,"(-) Subconta - Ajuste Valor Presente – Imobilizado","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo dos ativos. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até a revenda ou baixa do ativo, quando valor do AVP poderá ser excluído do Lucro Real do período (art.5º, Lei nº 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102030576,1.02.03.05.76,"Subconta - Ajuste Valor Presente – Depreciação, Amort., Exaustão Acumulada – Outros Imobilizados","Contas que registram a depreciação/amortização/exaustão acumulada sobre o ajuste a valor presente registrado na subconta 1.02.03.05.75. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102030590,1.02.03.05.90,"Subconta – Adoção Inicial - Outros Imobilizados","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF Nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102030591,1.02.03.05.91,"Subconta – Adoção Inicial – Depreciação, Amort., Exaustão Acumulada – Outros Imobilizados","Contas que registram a depreciação/amortização/exaustão acumulada sobre o ajuste adoção inicial registrado na subconta 1.02.03.05.90. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102030592,1.02.03.05.92,"Subconta – Adoção Inicial – Amortização Acumulada - Intangível|","Contas que registram a depreciação/amortização/exaustão acumulada sobre o ajuste adoção inicial registrado na subconta 1.02.03.05.90. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102030595,1.02.03.05.95,"Subconta – Adoção Inicial – Taxa Depreciação Diferente - Outros Imobilizados","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor da depreciação acumulada mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Esta conta registra apenas a diferença gerada, durante a vigência do RTT, pelo uso de taxas de depreciação diferentes das definidas nos anexos I e II da Instrução Normativa SRF nº 162/1998. Detalhes sobre a contabilização estão descritos no Anexo IV, Instrução Normativa SRF nº 1.515/2014). Após a adoção da Lei nº 12.973/2014 estas diferenças posteriormente geradas passarão a ser controladas exclusivamente no eLalur(art. 168, Instrução Normativa SRF nº 1.515/2014). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102050101,1.02.05.01.01,"Marcas","Contas que registram os custos de aquisição e registro de marcas, bem como desembolso a terceiros por contrato de uso. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC 04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.",asset_non_current,,l10n_br_account_chart_template
account_template_102050102,1.02.05.01.02,"Patentes e Segredos Industriais","Contas que registram os custos de aquisição e registro de patentes, bem como desembolso a terceiros por contrato de uso. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC 04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.Conforme itens 54/56, CPC 04, os gastos com pesquisa devem ser reconhecidos como despesa, enquando os despendidos no desenvolvimento devem ser ativados, itens 57/59.",asset_non_current,,l10n_br_account_chart_template
account_template_102050103,1.02.05.01.03,"Direitos de Exploração de Serviços Públicos","Contas que registram o ativo intangível representativo do direito de exploração de serviços públicos, é constituído durante a fase de contrução à medida em que recebe o direito(autorização) de cobrar os usuários dos serviços públicos( item 17, nos termos do ICPC 01(R1)). Se os serviços de construção do concessionário são pagos parte em ativo financeiro e parte em ativo intangível, é necessário contabilizar cada componente da remuneração separadamente. A remuneração recebida ou a receber de ambos os componentes deve ser inicialmente registrada pelo seu valor justo recebido ou a receber. Importante ressaltar que a natureza da remuneração deve ser determinada de acordo com os termos do contrato e, quando houver, legislação aplicável. Em termos gerais, este intangível é formado ao longo da fase de construção pela contrapartida das parcelas da receita de construção de cada contrato(conta 3.01.01.01.01.20), reconhecidas pelo método da porcentagem completada (itens 25/26, CPC 17(R1)). Nos termos do CPC 20, os custos de empréstimos atribuíveis ao contrato de concessão devem ser capitalizados durante a fase de construção, integrando o custo do intangível. Regras especiais devem ser aplicadas às concessões onerosas-direito de outorga, conforme OCPC 05. Referido intangível deve ser amortizado dentro do prazo da concessão. Para fins fiscais, o resultado apurado durante a fase construção deverá ser excluído do Lucro Real, controlado na parte B do eLalur, para ser adicionado na proporção em que o ativo intangível for realizado(art. 82, Instrução Normativa SRF nº 1.515/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102050104,1.02.05.01.04,"Direitos de Exploração de Recursos Florestais","Contas que registram os custos com aquisição dos direitos de exploração de recursos florestais, nos termos do ICPC 01.",asset_non_current,,l10n_br_account_chart_template
account_template_102050105,1.02.05.01.05,"Direitos de Exploração de Recursos Minerais","Contas que registram os custos com aquisição dos direitos de exploração de recursos minerais. Nos termos do CPC04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.",asset_non_current,,l10n_br_account_chart_template
account_template_102050106,1.02.05.01.06,"Direitos de Exploração de Recursos Hídricos","Contas que registram os custos com aquisição dos direitos de exploração de recursos hídricos. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.",asset_non_current,,l10n_br_account_chart_template
account_template_102050107,1.02.05.01.07,"Direitos Autorais","Contas que registram os custos com aquisição de direitos autorais. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC 04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.Conforme itens 54/56, CPC 04, os gastos com pesquisa devem ser reconhecidos como despesa, enquando os despendidos no desenvolvimento devem ser ativados, itens 57/59.",asset_non_current,,l10n_br_account_chart_template
account_template_102050108,1.02.05.01.08,"Patrimônio Cultural","Contas que registram os custos com aquisição de patrimônio cultural. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC 04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.",asset_non_current,,l10n_br_account_chart_template
account_template_102050109,1.02.05.01.09,"Fundo de Comércio","Contas que registram os custos com aquisição de fundos de comércio. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC 04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.",asset_non_current,,l10n_br_account_chart_template
account_template_102050110,1.02.05.01.10,"Software ou Programas de Computador","Contas que registram os custos com aquisição e/ou desenvolvimento de softwares. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC 04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.Conforme itens 54/56, CPC 04, os gastos com pesquisa devem ser reconhecidos como despesa, enquando os despendidos no desenvolvimento devem ser ativados, itens 57/59.",asset_non_current,,l10n_br_account_chart_template
account_template_102050111,1.02.05.01.11,"Contratos de Aluguel","Contas que registram os custos com contratos de aluguel. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC 04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.",asset_non_current,,l10n_br_account_chart_template
account_template_102050112,1.02.05.01.12,"Contratos de Franquias","Contas que registram os custos com aquisição de franquias. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC 04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.",asset_non_current,,l10n_br_account_chart_template
account_template_102050113,1.02.05.01.13,"Desenvolvimento de Produtos ou Serviços","Contas que registram os custos com o desenvolvimento de novos produtos ou serviços, que atendam aos requisitos de viabilidade técnica, intenção e capacidade uso e/ou venda. Nos termos do CPC 04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC 04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.Conforme itens 54/56, CPC 04, os gastos com pesquisa devem ser reconhecidos como despesa, enquando os despendidos no desenvolvimento devem ser ativados, itens 57/59.",asset_non_current,,l10n_br_account_chart_template
account_template_102050114,1.02.05.01.14,"Direito Readquirido","Contas que registram os direito readquirido em combinação de negócios, conforme item B35, CPC 15(R1).",asset_non_current,,l10n_br_account_chart_template
account_template_102050115,1.02.05.01.15,"Leasing Operacional Contratado pela Adquirida em Condições Mais Favoráveis","Contas que registram os direitos adquiridos em combinação de negócios, sobre contratos de leasing operacional contratados pela adquirida em condições mais favoráveis, conforme item B29, CPC 15(R1).",asset_non_current,,l10n_br_account_chart_template
account_template_102050116,1.02.05.01.16,"Intangíveis Não Reconhecidos na Adquirida","Contas que registram os intangíveis adquiridos em combinação de negócios, não reconhecidos na adquirida, conforme item 13, CPC15(R1).",asset_non_current,,l10n_br_account_chart_template
account_template_102050117,1.02.05.01.17,"Intangíveis Recebidos em Subvenções Governamentais","Contas que registram intangíveis recebidos em contrapartida a subvenções governamentais, conforme item 44, CPC 04.",asset_non_current,,l10n_br_account_chart_template
account_template_102050118,1.02.05.01.18,"(-) Redutora de Intangíveis Recebidos em Subvenções Governamentais","Contas redutoras de intangíveis recebidos em subvenções governamentais.",asset_non_current,,l10n_br_account_chart_template
account_template_102050120,1.02.05.01.20,"(-) Amortização Acumulada - Intangível","Contas que registram amortização das contas do ativo intangível.",asset_non_current,,l10n_br_account_chart_template
account_template_102050121,1.02.05.01.21,"Goodwill – Intangível","Contas que registram goodwill gerado em outras modalidades de combinações de negócio previstas no CPC 15, exceto aquisições de participação societárias registradas nas contas 1.02.02.01.10 e 1.02.02.02.10, bem como, em eventos societários de incorporação, fusão e cisão. Nos termos do item 48, CPC 04, goodwill gerado internamente não deve ser reconhecido como ativo.",asset_non_current,,l10n_br_account_chart_template
account_template_102050128,1.02.05.01.28,"Outros Intangíveis","Contas que registram os custos com aquisição de outros itens classificáveis no intangível. Nos termos do CPC04, ativo intangível é um ativo não monetário identificável sem substância física, ou seja, regra geral, para registrá-los no ativo necessário que sejam identificáveis, controlados e geradores de benefícios econômicos futuros. Conforme itens 63/64, CPC04, os gastos incorridos com marcas, títulos de publicação, lista de clientes e outros itens similares, quando gerados internamente, não devem ser reconhecidos como ativos intangíveis. Após o seu reconhecimento inicial, um ativo intangível deve ser apresentado ao custo, menos a eventual amortização acumulada e a perda acumulada do valor recuperável. Em se definindo sua vida útil, este bem deve ser amortizado por este prazo, o mesmo não ocorrendo para aqueles com vida útil indefinida, ou seja, com base na análise de todos os fatores relevantes, não existe um limite previsível para o período durante o qual o ativo deverá gerar fluxos de caixa líquidos positivos para a pessoa jurídica.Conforme itens 54/56, CPC04, os gastos com pesquisa devem ser reconhecidos como despesa, enquando os despendidos no desenvolvimento devem ser ativados, itens 57/59.",asset_non_current,,l10n_br_account_chart_template
account_template_102050129,1.02.05.01.29,"(-) Outras Contas Redutoras do Intangível","Outras contas redutoras do intangível, não classificáveis em outras contas específicas.",asset_non_current,,l10n_br_account_chart_template
account_template_102050155,1.02.05.01.55,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Intangível","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil do ativo intangível. Esta conta também registra as eventuais reversões. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 32, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102050156,1.02.05.01.56,"(-) Perdas por Redução ao Valor Recuperável (Impairment) do Goodwill - Intangível","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil do goodwill, exceto resultante de aquisição de participação societária, cujas perdas por desvalorização estão registrados nas contas 1.02.02.01.56 e 1.02.02.02.56. Estes valores não estão sujeitos a reversão, conforme item 124, CPC 01. Referidos valores deverão ser adicionados ao Lucro Real, mantidos na Parte B do eLalur até ocorrência da alienação ou baixa do ativo, quando poderão ser excluídos do Lucro Real(art. 28, Lei nº 12.973/2014).",asset_non_current,,l10n_br_account_chart_template
account_template_102050175,1.02.05.01.75,"( - ) Subconta – Ajuste a Valor Presente (AVP) - Intangível","Contas que registram contrapartidas dos ajustes a valor presente efetuados sobre os passivos decorrentes de compras a prazo de intangíveis. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real. Concomitante controle ocorrerá nesta subconta até que bem seja realizado, quando valor do AVP poderá ser excluído do Lucro Real do período(art.5º, Lei 12.973/2014). Detalhes sobre a contabilização estão descritos no Anexo I, exemplo 4, Instrução Normativa SRF nº 1.515/2014). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102050177,1.02.05.01.77,"Subconta - Ajuste Valor Presente – Amortização Acumulada - Intangível","Contas que registram a amortização acumulada sobre o ajuste a valor presente registrado na subconta 1.02.05.01.75. A operacionalização desta subconta está evidenciada no exemplo 4, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102050190,1.02.05.01.90,"Subconta – Adoção Inicial - Intangível","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida da realização do bem, (arts. 164 e 167, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014). No caso do ativo não estar reconhecido no FCONT, apenas na contabilidade societária, fica dispensada a constituição da subconta de adoção inicial, nos termos do art. 169, §3º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_102050192,1.02.05.01.92,"Subconta – Adoção Inicial – Amortização Acumulada - Intangível","Contas que registram a amortização acumulada sobre o ajuste adoção inicial registrado na subconta 1.02.05.01.90. A operacionalização desta subconta está evidenciada nos exemplos 1/2, anexo I, da Instrução Normativa SRF nº 1.515/2014",asset_non_current,,l10n_br_account_chart_template
account_template_102060101,1.02.06.01.01,"Despesas Pré-Operacionais ou Pré-Industriais – Ativo Diferido","Contas que registram os gastos de organização e administração, encargos financeiros líquidos, estudos, projetos e detalhamentos, juros a acionista na fase de implantação e gastos preliminares de operação. O saldo existente em 31 de dezembro de 2008 no ativo diferido que, pela sua natureza, não puder ser alocado a outro grupo de contas, poderá permanecer no ativo sob essa classificação até sua completa amortização, sujeito à análise sobre a recuperação.",asset_non_current,,l10n_br_account_chart_template
account_template_102060102,1.02.06.01.02,"Despesas com Pesquisas Científicas ou Tecnológicas – Ativo Diferido","Contas que registram os gastos com pesquisa científica ou tecnológica. O saldo existente em 31 de dezembro de 2008 no ativo diferido que, pela sua natureza, não puder ser alocado a outro grupo de contas, poderá permanecer no ativo sob essa classificação até sua completa amortização, sujeito à análise sobre a recuperação.",asset_non_current,,l10n_br_account_chart_template
account_template_102060103,1.02.06.01.03,"Demais Aplicações em Despesas Amortizáveis – Ativo Diferido","Contas que registram os gastos com pesquisas e desenvolvimento de produtos, com a implantação de sistemas e métodos e com reorganização. O saldo existente em 31 de dezembro de 2008 no ativo diferido que, pela sua natureza, não puder ser alocado a outro grupo de contas, poderá permanecer no ativo sob essa classificação até sua completa amortização, sujeito à análise sobre a recuperação.",asset_non_current,,l10n_br_account_chart_template
account_template_102060131,1.02.06.01.31,"(-) Amortização Acumulada - Ativo Diferido","Contas que registram a amortização das contas do ativo diferido.",asset_non_current,,l10n_br_account_chart_template
account_template_102060156,1.02.06.01.56,"(-) Perdas por Redução ao Valor Recuperável (Impairment) - Ativo Diferido","Contas que registram as perdas estimadas com base em evidências objetivas que demonstrem a não recuperabilidade do valor contábil do ativo diferido. Esta conta também registra as eventuais reversões.",asset_non_current,,l10n_br_account_chart_template
account_template_102060190,1.02.06.01.90,"Subconta – Adoção Inicial – Ativo Diferido","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do ativo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014). No caso de ativo diferido existente apenas no FCONT, não haverá registro em subcontas, devendo tal valor ser controlado na parte B do eLalur(art. 171, §1º, Instrução Normativa SRF nº 1.515/2014). No caso do ativo não estar reconhecido na contabilidade societária, apenas no FCONT, a diferença em questão deverá ser controlada na parte B do eLalur, nos termos do art. 171, §1º, Instrução Normativa SRF nº 1.515/2014)",asset_non_current,,l10n_br_account_chart_template
account_template_201010101,2.01.01.01.01,"Salários e Remunerações a Pagar","Contas que registram o valor correspondente aos salários, ordenados, pro labore, horas extras, , adicionais e prêmios a serem pagos no exercício subsequente, inclusive gratificações a empregados, o valor das férias do período aquisitivo já completo, 13 salários a pagar.",liability_current,,l10n_br_account_chart_template
account_template_201010102,2.01.01.01.02,"Participações no Resultado a Pagar","Contas que registram o valor correspondente a participação no resultado a serem pagos no exercício subsequente de administradores e empregados.",liability_current,,l10n_br_account_chart_template
account_template_201010103,2.01.01.01.03,"INSS a Recolher","Contas que registram o valor das Contribuições Previdenciárias a recolher.",liability_current,,l10n_br_account_chart_template
account_template_201010104,2.01.01.01.04,"FGTS a Recolher","Contas que registram o valor do FGTS a recolher.",liability_current,,l10n_br_account_chart_template
account_template_201010105,2.01.01.01.05,"Benefícios Não Monetários","Contas que registram os benefícios não monetários a pagar. Nos termos do item 9(d), CPC 33(R1), benefícios de curto prazo tais como assistência médica, moradia, carros e bens ou serviços gratuitos ou subsidiados para os atuais empregados ou administradores.",liability_current,,l10n_br_account_chart_template
account_template_201010150,2.01.01.01.50,"Benefícios Pós-Emprego","Contas que registram os benefícios pós-emprego a pagar. Nos termos do item 26, CPC 33(R1), aqueles pagos após o período de emprego, tais como aposentadoria, pensões, seguro de via e assistência médica pós-emprego.",liability_current,,l10n_br_account_chart_template
account_template_201010155,2.01.01.01.55,"Outros Benefícios de Longo Prazo","Contas que registram outros benefícios de longo prazo a pagar no curto prazo. Nos termos do item 5(c), CPC 33(R1), são aqueles que não se espera sejam integralmente liquidados em até doze meses após o fim do exercício em que os empregados prestarem o respectivo serviço, tais como ausências remuneradas de longo prazo, licenças por tempo de serviço, jubileu , benefícios por invalidez de longo prazo.",liability_current,,l10n_br_account_chart_template
account_template_201010160,2.01.01.01.60,"Benefícios Rescisórios","Contas que registram os benefícios rescisórios a pagar. Nos termos do item 8, CPC 33(R1), são aqueles fornecidos pela rescisão do contrato de trabalho de empregado, seja decisão da pessoa jurídica de terminar o vínculo empregatício do empregado antes da data normal da aposentadoria, seja decisão do empregado de aceitar uma oferta de benefícios em troca da rescisão do contrato de trabalho.",liability_current,,l10n_br_account_chart_template
account_template_201010109,2.01.01.01.09,"Demais Encargos a Recolher","Contas correspondentes a outros encargos a recolher incidentes sobre a folha de pagamentos dos funcionários, exceto INSS e FGTS.",liability_current,,l10n_br_account_chart_template
account_template_201010301,2.01.01.03.01,"Fornecedores - Operações com Partes Não Relacionadas - No País – Circulante","Contas que registram o valor a pagar correspondentes à compra de bens, direitos e serviços de fornecedores nacionais, não relacionados com declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12",liability_payable,TRUE,l10n_br_account_chart_template
account_template_201010302,2.01.01.03.02,"Fornecedores - Operações com Partes Não Relacionadas - No Exterior – Circulante","Contas que registram o valor a pagar correspondentes à compra de de bens, direitos e serviços de fornecedores estrangeiros, não relacionados com declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12",liability_payable,TRUE,l10n_br_account_chart_template
account_template_201010303,2.01.01.03.03,"Fornecedores - Operações com Partes Relacionadas - No País – Circulante","Contas que registram o valor a pagar correspondentes à compra de bens, direitos e serviços de fornecedores nacionais, relacionados com declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12",liability_payable,TRUE,l10n_br_account_chart_template
account_template_201010304,2.01.01.03.04,"Fornecedores - Operações com Partes Relacionadas - No Exterior – Circulante","Contas que registram o valor a pagar correspondentes à compra de bens, direitos e serviços de fornecedores estrangeiros, relacionados com declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12",liability_payable,TRUE,l10n_br_account_chart_template
account_template_201010350,2.01.01.03.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Fornecedores Circulante","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados nos instrumentos de dívidas vinculados a compras efetuadas a prazo. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Em sendo as contrapartidas registradas em subcontas vinculadas aos ativos adquiridos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_current,,l10n_br_account_chart_template
account_template_201010390,2.01.01.03.90,"Subconta – Adoção Inicial - Fornecedores - Circulante","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do passivo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida que o passivo for baixado ou liquidado (arts. 165 e 168, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2) Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o passivo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",liability_current,,l10n_br_account_chart_template
account_template_201010501,2.01.01.05.01,"Adiantamentos de Clientes - no País","Contas que registram o valor correspondente a adiantamentos de clientes no país.",liability_payable,TRUE,l10n_br_account_chart_template
account_template_201010502,2.01.01.05.02,"Adiantamentos de Clientes - no Exterior","Contas que registram o valor correspondente a adiantamentos de clientes no exterior.",liability_payable,TRUE,l10n_br_account_chart_template
account_template_201010550,2.01.01.05.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Contas a Pagar - Circulante","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados nos instrumentos de dívidas vinculados a adiantamentos de clientes. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Em sendo as contrapartidas registradas em subcontas vinculadas a ativos adquiridos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_payable,TRUE,l10n_br_account_chart_template
account_template_201010590,2.01.01.05.90,"Subconta – Adoção Inicial - Contas a Pagar - Circulante","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do passivo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida que o passivo for baixado ou liquidado (arts. 165 e 168, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o passivo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",liability_payable,TRUE,l10n_br_account_chart_template
account_template_201010701,2.01.01.07.01,"Duplicatas Descontadas – Circulante","Contas que registram o valor das parcelas a serem subtraídas do circulante, correspondentes a valores das duplicatas descontadas que retificam o grupo de clientes. Ainda que represente uma dívida, caso a pessoa jurídica registre esta conta no ativo, deve ser classificada neste referencial junto com a conta que retifique na contabilidade societária.",liability_current,,l10n_br_account_chart_template
account_template_201010702,2.01.01.07.02,"Empréstimos ou Financiamentos - no País - Circulante","Contas que registram o valor dos financiamentos e empréstimos a curto prazo, obtidos com instituição financeira no Brasil. Encargos financeiros a transcorrer e juros a pagar decorrentes devem ser classificadas nesta conta. As obrigações por empréstimos tomados com pessoa jurídica não financeira e física deverão ser informados na conta de mútuos.",liability_current,,l10n_br_account_chart_template
account_template_201010703,2.01.01.07.03,"Empréstimos ou Financiamentos - no Exterior – Circulante","Contas que registram o valor dos financiamentos e empréstimos a curto prazo, obtidos com instituição financeira no Exterior. Encargos financeiros a transcorrer e juros a pagar decorrentes devem ser classificadas nesta conta.As obrigações por empréstimos tomados com pessoa jurídica não financeira e física deverão ser informados na conta de mútuos.",liability_current,,l10n_br_account_chart_template
account_template_201010704,2.01.01.07.04,"Adiantamentos de Contrato de Câmbio – Circulante","Contas que registram o valor das obrigações de curto prazo relativas às operações de crédito na modalidade de adiantamento de contrato de câmbio.",liability_current,,l10n_br_account_chart_template
account_template_201010705,2.01.01.07.05,"Arrendamento - no País – Circulante","Contas que registram o valor das obrigações de curto prazo relativas a arrendamento contratados no país.",liability_current,,l10n_br_account_chart_template
account_template_201010706,2.01.01.07.06,"Arrendamento - no Exterior - Circulante","Contas que registram o valor das obrigações de curto prazo relativas a arrendamento contratados no exterior.",liability_current,,l10n_br_account_chart_template
account_template_201010750,2.01.01.07.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Empréstimos e Financiamentos - Circulante","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados nos instrumentos de dívidas vinculados a empréstimos e financiamentos. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Exceto para as operações de leasing financeiro(art. 89, Instrução Normativa SRF nº 1.515/2014) em sendo as contrapartidas registradas em subcontas vinculadas aos ativos adquiridos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_current,,l10n_br_account_chart_template
account_template_201010790,2.01.01.07.90,"Subconta – Adoção Inicial - Empréstimos e Financiamentos - Circulante","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do passivo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida que o passivo for baixado ou liquidado (arts. 165 e 168, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o passivo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",liability_current,,l10n_br_account_chart_template
account_template_201010901,2.01.01.09.01,"IRRF a Recolher – Circulante","Contas que registram o valor referentes ao IRRF a Recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010902,2.01.01.09.02,"IPI a Recolher – Circulante","Contas que registram o valor referentes ao IPI a Recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010903,2.01.01.09.03,"ICMS a Recolher – Circulante","Contas que registram o valor referente ao ICMS a Recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010904,2.01.01.09.04,"PIS a Recolher – Circulante","Contas que registram o valor do PIS a recolher.",liability_current,,l10n_br_account_chart_template
account_template_201010905,2.01.01.09.05,"COFINS a Recolher – Circulante","Contas que registram o valor da COFINS a recolher.",liability_current,,l10n_br_account_chart_template
account_template_201010906,2.01.01.09.06,"IOF a Recolher – Circulante","Contas que registram o valor do IOF a recolher.",liability_current,,l10n_br_account_chart_template
account_template_201010907,2.01.01.09.07,"CIDE a Recolher – Circulante","Contas que registram o valor da CIDE a recolher.",liability_current,,l10n_br_account_chart_template
account_template_201010908,2.01.01.09.08,"Tributos Municipais a Recolher – Circulante","Contas que registram o valor dos tributos municipais a Recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010909,2.01.01.09.09,"Parcelamentos Especiais a Recolher - Tributos Federais – Circulante","Contas que registram o valor dos saldos de parcelamentos especiais e ordinários de tributos federais a Recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010910,2.01.01.09.10,"Parcelamentos Especiais a Recolher - Tributos Estaduais e Municipais – Circulante","Contas que registram o valor dos saldos de parcelamentos especiais e ordinários de tributos estaduais e municipais a recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010911,2.01.01.09.11,"Contribuições Previdenciárias a Recolher – Desoneração da Folha de Pagamento – Circulante","Contas que registram o valor referente à contribuição sobre o faturamento em substituição ao INSS sobre a folha. Informar o saldo a recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010912,2.01.01.09.12,"Tributos Retidos a Recolher – Circulante","Contas que registram o valor referente aos tributos retidos a recolher. Informar o saldo a recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010913,2.01.01.09.13,"IRPJ a Recolher – Circulante","Contas que registram o valor referente ao IRPJ a recolher. Informar o saldo a recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010914,2.01.01.09.14,"CSLL a Recolher – Circulante","Contas que registram o valor referente à CSLL a recolher. Informar o saldo a recolher no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201010928,2.01.01.09.28,"Outros Tributos a Recolher – Circulante","Contas que registram o valor dos tributos a recolher não classificáveis em contas específicas.",liability_current,,l10n_br_account_chart_template
account_template_201011101,2.01.01.11.01,"Derivativos - Hedge Valor Justo - No País","Contas que registram os instrumentos destinados a hedge de valor justo operados em ambiente negocial no país. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",liability_current,,l10n_br_account_chart_template
account_template_201011102,2.01.01.11.02,"Derivativos - Hedge Fluxo de Caixa - No País","Contas que registram os instrumentos destinados a hedge de fluxo de caixa operados em ambiente negocial no país. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",liability_current,,l10n_br_account_chart_template
account_template_201011103,2.01.01.11.03,"Derivativos - Hedge Investimento no Exterior - No País","Contas que registram os instrumentos destinados a hedge de fluxo de investimento no exterior operados em ambiente negocial no país. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",liability_current,,l10n_br_account_chart_template
account_template_201011170,2.01.01.11.70,"Subconta - Ajuste a Valor Justo - Valores Mobiliários – Hedge - No País","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os passivos financeiros, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da baixa ou liquidação do passivo(arts. 49/53, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",liability_current,,l10n_br_account_chart_template
account_template_201011190,2.01.01.11.90,"Subconta – Adoção Inicial - Valores Mobiliários – Hedge - No País","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do passivo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida que o passivo for baixado ou liquidado (arts. 165 e 168, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o passivo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",liability_current,,l10n_br_account_chart_template
account_template_201011201,2.01.01.12.01,"Derivativos - Hedge Valor Justo - No Exterior","Contas que registram os instrumentos destinados a hedge de valor justo operados em ambiente negocial no exterior. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",liability_current,,l10n_br_account_chart_template
account_template_201011202,2.01.01.12.02,"Derivativos - Hedge Fluxo de Caixa - No Exterior","Contas que registram os instrumentos destinados a hedge de fluxo de caixa operados em ambiente negocial no exterior. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",liability_current,,l10n_br_account_chart_template
account_template_201011203,2.01.01.12.03,"Derivativos - Hedge Investimento no Exterior - No Exterior","Contas que registram os instrumentos destinados a hedge de fluxo de investimento no exterior operados em ambiente negocial no exterior. Toda documentação exigida para operar em “Hedge Accounting” deve constar em Nota Explicativa encaminhada pela ECD.",liability_current,,l10n_br_account_chart_template
account_template_201011270,2.01.01.12.70,"Subconta - Ajuste a Valor Justo - Valores Mobiliários – Hedge - No Exterior","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os passivos financeiros, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da baixa ou liquidação do passivo(arts. 49/53, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6), sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF Nº 213/2002. Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",liability_current,,l10n_br_account_chart_template
account_template_201011290,2.01.01.12.90,"Subconta – Adoção Inicial - Valores Mobiliários – Hedge - No Exterior","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do passivo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida que o passivo for baixado ou liquidado (arts. 165 e 168, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2), sem prejuízo observância aos arts. 9º a 12 da Instrução Normativa SRF Nº 213/2002. Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o passivo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",liability_current,,l10n_br_account_chart_template
account_template_201011301,2.01.01.13.01,"Debêntures a Pagar – Circulante","Contas que registram o valor das debêntures a pagar no final do período de apuração. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011302,2.01.01.13.02,"Prêmio na Emissão de Debêntures – Circulante","Contas que registram o valor do prêmio na emissão de debêntures a pagar no final do período de apuração",liability_current,,l10n_br_account_chart_template
account_template_201011304,2.01.01.13.04,"Notas Promissórias a Pagar","Contas que registram o valor notas promissórias(“commercial papers”) a pagar no final do período de apuração. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011305,2.01.01.13.05,"Bonds a Pagar","Contas que registram o valor de Bonds a pagar no final do período de apuração. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011306,2.01.01.13.06,"Certificados de Recebíveis Imobiliários - CRI","Contas que registram o valor de Certificados de Recebíveis Imobiliários a pagar no final do período de apuração. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011307,2.01.01.13.07,"Certificados de Recebíveis do Agronegócio - CRA","Contas que registram o valor de Certificados de Recebíveis do Agronegócio a pagar no final do período de apuração. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011325,2.01.01.13.25,"Outros Títulos de Dívida a Pagar – Pelo Custo Amortizado - Circulante","Contas que registram outros títulos emitidos para a captação de recursos financeiros, não classificados em contas mais específicas, mensurados pelo custo amortizado. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011327,2.01.01.13.27,"(-) Custos a Amortizar – Títulos de Dívida - Circulante","Contas que registram o valor do custo a amortizar dos títulos de dívida, saldo existente no final do período de apuração.",liability_current,,l10n_br_account_chart_template
account_template_201011328,2.01.01.13.28,"Outros Títulos de Dívida a Pagar – Pelo Pelo Valor Justo(VJPR) - Circulante","Contas que registram outros títulos emitidos para a captação de recursos financeiros, não classificados em contas mais específicas, mensurados pelo valor justo reconhecido diretamente no resultado.",liability_current,,l10n_br_account_chart_template
account_template_201011329,2.01.01.13.29,"(-) Deságio a Apropriar – Títulos de Dívida - Circulante","Contas que registram o valor do deságio aplicado a títulos de dívida.",liability_current,,l10n_br_account_chart_template
account_template_201011350,2.01.01.13.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Títulos de Dívida - Circulante","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados nos títulos de dívidas. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Em sendo as contrapartidas registradas em subcontas vinculadas a ativos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_current,,l10n_br_account_chart_template
account_template_201011370,2.01.01.13.70,"Subconta - Ajuste a Valor Justo – Títulos de Dívida a Pagar - Circulante","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os passivos financeiros, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da baixa ou liquidação do passivo(arts. 44 e 48, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",liability_current,,l10n_br_account_chart_template
account_template_201011390,2.01.01.13.90,"Subconta – Adoção Inicial - Títulos de Dívida - Circulante","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do passivo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida que o passivo for baixado ou liquidado (arts. 165 e 168, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o passivo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",liability_current,,l10n_br_account_chart_template
account_template_201011501,2.01.01.15.01,"Provisão para o Imposto de Renda","Contas que registram o valor da provisão para o imposto de renda a pagar.",liability_current,,l10n_br_account_chart_template
account_template_201011502,2.01.01.15.02,"Provisão para a Contribuição Social sobre o Lucro Líquido","Contas que registram o valor da provisão para a contribuição social sobre o lucro líquido a pagar.",liability_current,,l10n_br_account_chart_template
account_template_201011503,2.01.01.15.03,"Férias a Pagar","Contas que registram o valor de férias a pagar de administradores e empregados.",liability_current,,l10n_br_account_chart_template
account_template_201011504,2.01.01.15.04,"13º Salário a Pagar","Contas que registram o valor de 13º salário a pagar de administradores e empregados.",liability_current,,l10n_br_account_chart_template
account_template_201011505,2.01.01.15.05,"Provisões de Natureza Trabalhista - Circulante","Contas que registram o valor da provisão de natureza trabalhista.",liability_current,,l10n_br_account_chart_template
account_template_201011506,2.01.01.15.06,"Provisões de Natureza Tributária – Circulante","Contas que registram o valor da provisão de natureza tributária.",liability_current,,l10n_br_account_chart_template
account_template_201011507,2.01.01.15.07,"Provisões de Natureza Cível – Circulante","Contas que registram o valor da provisão de natureza cível.",liability_current,,l10n_br_account_chart_template
account_template_201011528,2.01.01.15.28,"Outras Provisões","Contas que registram o valor das demais provisões não classificadas em contas mais específicas.",liability_current,,l10n_br_account_chart_template
account_template_201011550,2.01.01.15.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Provisões - Circulante","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados nas provisões. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Em sendo as contrapartidas registradas em subcontas vinculadas a ativos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_current,,l10n_br_account_chart_template
account_template_201011701,2.01.01.17.01,"Mútuos – Partes Não Relacionadas – No País - Ciculante","Contas que registram os empréstimos tomados de partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, situadas no país. Juros e encargos decorrentes também devem ser registrados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011702,2.01.01.17.02,"Mútuos - Partes Não Relacionadas – No Exterior - Ciculante","Contas que registram os empréstimos tomados de partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, situadas no exterior. Juros e encargos decorrentes também devem ser registrados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011703,2.01.01.17.03,"Mútuos – Partes Relacionadas – No País – Circulante","Contas que registram os empréstimos tomados de partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, situadas no país. Juros e encargos decorrentes também devem ser registrados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011704,2.01.01.17.04,"Mútuos - Partes Relacionadas – No Exterior - Circulante","Contas que registram os empréstimos tomados de partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, situadas no exterior. Juros e encargos decorrentes também devem ser registrados nesta conta.",liability_current,,l10n_br_account_chart_template
account_template_201011709,2.01.01.17.09,"Contraprestação Contingente Passiva - Combinação de Negócios - Circulante","Contas que registram a contraprestação contingente passiva, em uma combinação de negócios. Em termos gerais, constitui cláusula assumida pelo adquirente de transferir ativos ou participações societárias adicionais , aos ex-proprietários da adquirida, se certas condições específicas venham a ocorrer. Para fins de gerar efeito sobre tratamento fiscal das parcelas integrantes do custo de aquisição de participação societária, deve-se observar os arts. 110/111, Instrução Normativa SRF Nº 1.515/2014 .",liability_current,,l10n_br_account_chart_template
account_template_201011710,2.01.01.17.10,"Passivo Contingente Assumido em Combinação de Negócios - Circulante","Contas que registram o passivo contingente assumido de curto prazo em uma combinação de negócios.",liability_current,,l10n_br_account_chart_template
account_template_201011711,2.01.01.17.11,"Faturamento para Entrega Futura - Circulante","Contas que registram os faturamentos antecipados, por conta de futuros fornecimentos, não gerando nenhum direito, portanto retificando o saldo de duplicatas a receber.",liability_current,,l10n_br_account_chart_template
account_template_201011712,2.01.01.17.12,"Juros sobre o Capital Próprio a Pagar - Circulante","Contas que registram o valor dos juros sobre o capital próprio a serem pagos no exercício subsequente aos sócios ou acionistas.",liability_current,,l10n_br_account_chart_template
account_template_201011713,2.01.01.17.13,"Dividendos a Pagar – Circulante","Contas que registram o valor dos dividendos aprovados pela Assembleia, creditados aos acionistas ou propostos pela administração da pessoa jurídica na data do balanço, como parte da destinação proposta para os lucros.",liability_current,,l10n_br_account_chart_template
account_template_201011715,2.01.01.17.15,"Conta de Controle de Custo Contratado - Circulante","Contas que registram o controle de custo contratado, utilizadas pelas pessoas jurídicas da atividade imobiliária.",liability_current,,l10n_br_account_chart_template
account_template_201011716,2.01.01.17.16,"Conta de Controle de Custo Orçado - Circulante","Contas que registram o controle de custo orçado, utilizadas pelas pessoas jurídicas da atividade imobiliária.",liability_current,,l10n_br_account_chart_template
account_template_201011725,2.01.01.17.25,"Direitos Creditórios a Pagar - Circulante","Contas que registram os direitos creditórios a pagar, utilizadas por pessoa jurídica que exerça ativididade de securitização.",liability_current,,l10n_br_account_chart_template
account_template_201011728,2.01.01.17.28,"Outras Obrigações – Circulante","Contas que registram outras obrigações na classificadas em contas mais específicas.",liability_current,,l10n_br_account_chart_template
account_template_201011750,2.01.01.17.50,"(-)Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Outras Contas a Pagar - Circulante","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados em outras obrigações. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Em sendo as contrapartidas registradas em subcontas vinculadas a ativos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_current,,l10n_br_account_chart_template
account_template_201011760,2.01.01.17.60,"CPC 47 - Passivos de Contrato - Circulante","Contas que registram os efeitos no passivo circulante decorrentes da adoção do Pronunciamento Técnico CPC 47 - Receita de Contrato com Cliente.",liability_current,,l10n_br_account_chart_template
account_template_201011901,2.01.01.19.01,"Receitas Diferidas","Contas que registram o valor das receitas faturadas e recebidas antecipadamente decorrente da venda de bens e serviços, cuja a execução e entrega ocorrem até o término do ano-calendário subsequente. Também se considera como receitas de exercícios futuros os juros e demais receitas financeiras recebidas antecipadamente em transações financeiras.",liability_current,,l10n_br_account_chart_template
account_template_201011902,2.01.01.19.02,"(-) Custos Correspondentes às Receitas Diferidas","Contas que registram o valor dos custos e despesas de exercícios futuros referentes às receitas diferidas.",liability_current,,l10n_br_account_chart_template
account_template_201011903,2.01.01.19.03,"Subvenção Governamental a Apropriar","Contas que registram o valor das doações e subvenções governamentais, enquanto não transferidas para o resultado do exercício.",liability_current,,l10n_br_account_chart_template
account_template_201011950,2.01.01.19.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Receitas Diferidas","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados em outras obrigações. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Em sendo as contrapartidas registradas em subcontas vinculadas a ativos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_current,,l10n_br_account_chart_template
account_template_202010101,2.02.01.01.01,"Fornecedores - No País - Longo Prazo","Contas que registram o valor a pagar correspondentes à compra de bens, direitos e serviços de fornecedores nacionais, seja relacionado ou não com declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",liability_non_current,,l10n_br_account_chart_template
account_template_202010102,2.02.01.01.02,"Fornecedores - No Exterior - Longo Prazo","Contas que registram o valor a pagar correspondentes à compra de bens, direitos e serviços de fornecedores estrangeiros, seja relacionado ou não com declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",liability_non_current,,l10n_br_account_chart_template
account_template_202010103,2.02.01.01.03,"Credores por Financiamento - Longo Prazo","Contas que registram obrigações resultantes de financiamentos de bens e equipamentos do ativo imobilizado concedidos à empresa pelos próprios fornecedores de tais bens.",liability_non_current,,l10n_br_account_chart_template
account_template_202010104,2.02.01.01.04,"Títulos a Pagar - Longo Prazo","Contas que registram outras títulos a pagar não classificados em contas mais específicas, principalmente no grupo 2.02.01.07",liability_non_current,,l10n_br_account_chart_template
account_template_202010105,2.02.01.01.05,"Duplicatas Descontadas - Longo Prazo","Contas que registram o valor das parcelas a serem subtraídas do longo prazo, correspondentes a valores das duplicatas descontadas que retificam o grupo de clientes. Ainda que represente uma dívida, caso a pessoa jurídica registre esta conta no ativo, deve ser classificada neste referencial junto com a conta que retifique na contabilidade societária.",liability_non_current,,l10n_br_account_chart_template
account_template_202010106,2.02.01.01.06,"Empréstimos ou Financiamentos - no País - Longo Prazo","Contas que registram o valor dos financiamentos e empréstimos a longo prazo, obtidos com instituição financeira no país. Encargos financeiros a transcorrer e juros a pagar decorrentes devem ser classificadas nesta conta. As obrigações por empréstimos tomados com pessoa jurídica não financeira e física deverão ser informados na conta de mútuos.",liability_non_current,,l10n_br_account_chart_template
account_template_202010107,2.02.01.01.07,"Empréstimos ou Financiamentos - no Exterior - Longo Prazo","Contas que registram o valor dos financiamentos e empréstimos a longo prazo, obtidos com instituição financeira no exterior. Encargos financeiros a transcorrer e juros a pagar decorrentes devem ser classificadas nesta conta. As obrigações por empréstimos tomados com pessoa jurídica não financeira e física deverão ser informados na conta de mútuos",liability_non_current,,l10n_br_account_chart_template
account_template_202010108,2.02.01.01.08,"Adiantamentos de Contrato de Câmbio - Longo Prazo","Contas que registram o valor das operações de crédito na modalidade de adiantamento de contrato de câmbio, vencíveis a longo prazo.",liability_non_current,,l10n_br_account_chart_template
account_template_202010109,2.02.01.01.09,"Arrendamento no País - Longo Prazo","Contas que registram o valor das obrigações relativas a arrendamento contratado no país, vencíveis a longo prazo.",liability_non_current,,l10n_br_account_chart_template
account_template_202010110,2.02.01.01.10,"Arrendamento no Exterior - Longo Prazo","Contas que registram o valor das obrigações relativas a arrendamento contratado no exterior, vencíveis a longo prazo.",liability_non_current,,l10n_br_account_chart_template
account_template_202010111,2.02.01.01.11,"Adiantamentos de Clientes - no País – Longo Prazo","Contas que registram o valor correspondente a adiantamentos de clientes no país de longo prazo.",liability_non_current,,l10n_br_account_chart_template
account_template_202010112,2.02.01.01.12,"Adiantamentos de Clientes - no Exterior – Longo Prazo","Contas que registram o valor correspondente a adiantamentos de clientes no exterior de longo prazo.",liability_non_current,,l10n_br_account_chart_template
account_template_202010150,2.02.01.01.50,"(-)Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Empréstimos e Financiamentos - Longo Prazo","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados nos instrumentos de dívidas vinculados a empréstimos e financiamentos. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Exceto para as operações de leasing financeiro(art. 89, Instrução Normativa SRF nº 1.515/2014) em sendo as contrapartidas registradas em subcontas vinculadas aos ativos adquiridos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_non_current,,l10n_br_account_chart_template
account_template_202010170,2.02.01.01.70,"Subconta - Ajuste a Valor Justo - Empréstimos e Financiamentos - Longo Prazo","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os passivos financeiros, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da baixa ou liquidação do passivo(arts. 49/53, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",liability_non_current,,l10n_br_account_chart_template
account_template_202010190,2.02.01.01.90,"Subconta – Adoção Inicial - Empréstimos e Financiamentos - Longo Prazo","Contas que registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do passivo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida que o passivo for baixado ou liquidado (arts. 165 e 168, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o passivo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",liability_non_current,,l10n_br_account_chart_template
account_template_202010201,2.02.01.02.01,"Benefícios Pós Emprego - Longo Prazo","Contas que registram os benefícios pós-emprego a pagar no longo prazo. Nos termos do item 26, CPC 33(R1), aqueles pagos após o período de emprego, tais como aposentadoria, pensões, seguro de via e assistência médica pós-emprego.",liability_non_current,,l10n_br_account_chart_template
account_template_202010202,2.02.01.02.02,"Outro Benefícios de Longo Prazo - Longo Prazo","Contas que registram outros benefícios de longo prazo a pagar no longo prazo. Nos termos do item 5(c), CPC 33(R1), são aqueles que não se espera sejam integralmente liquidados em até doze meses após o fim do exercício em que os empregados prestarem o respectivo serviço, tais como ausências remuneradas de longo prazo, licenças por tempo de serviço, jubileu , benefícios por invalidez de longo prazo.",liability_non_current,,l10n_br_account_chart_template
account_template_202010203,2.02.01.02.03,"Benefícios Rescisórios - Longo Prazo","Contas que registram os benefícios rescisórios a pagar no longo prazo. Nos termos do item 8, CPC 33(R1), são aqueles fornecidos pela rescisão do contrato de trabalho de empregado, seja decisão da pessoa jurídica de terminar o vínculo empregatício do empregado antes da data normal da aposentadoria, seja decisão do empregado de aceitar uma oferta de benefícios em troca da rescisão do contrato de trabalho.",liability_non_current,,l10n_br_account_chart_template
account_template_202010301,2.02.01.03.01,"Parcelamentos Especiais e Ordinários a Recolher - Tributos Federais - Longo Prazo","Contas que registram o valor de parcelamentos especiais e ordinários de tributos federais vencíveis a longo prazo.",liability_non_current,,l10n_br_account_chart_template
account_template_202010302,2.02.01.03.02,"Parcelamentos Especiais e Ordinários a Recolher - Tributos Estaduais e Municipais - Longo Prazo","Contas que registram o valor de parcelamentos especiais e ordinários de tributos estaduais e municipais vencíveis a longo prazo.",liability_non_current,,l10n_br_account_chart_template
account_template_202010328,2.02.01.03.28,"Outros Tributos a Recolher - Longo Prazo","Contas que registram o valor de outras obrigações tributárias vencíveis a longo prazo, não classificadas em contas mais específicas.",liability_non_current,,l10n_br_account_chart_template
account_template_202010501,2.02.01.05.01,"Débitos Fiscais IRPJ - Diferenças Temporárias - Longo Prazo","Contas que registram o valor do tributo devido em período futuro relacionado a diferenças temporárias tributáveis. Diferenças temporárias são divergências no valor contábil de ativo ou passivo no balanço e sua base fiscal, nos termos do CPC32.",liability_non_current,,l10n_br_account_chart_template
account_template_202010502,2.02.01.05.02,"Débitos Fiscais CSLL - Diferenças Temporárias - Longo Prazo","Contas que registram o valor do tributo devido em período futuro relacionado a diferenças temporárias tributáveis. Diferenças temporárias são divergências no valor contábil de ativo ou passivo no balanço e sua base fiscal, nos termos do CPC32.",liability_non_current,,l10n_br_account_chart_template
account_template_202010701,2.02.01.07.01,"Debêntures a Pagar - Longo Prazo","Contas que registram o valor das debêntures a pagar no longo prazo. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202010702,2.02.01.07.02,"Prêmio na Emissão de Debêntures - Longo Prazo","Contas que registram o valor do prêmio na emissão de debêntures a pagar no longo prazo. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202010704,2.02.01.07.04,"Notas Promissórias a Pagar – Longo Prazo","Contas que registram o valor notas promissórias(“commercial papers”) a pagar no longo prazo. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202010705,2.02.01.07.05,"Bonds a Pagar","Contas que registram o valor de Bonds a pagar no longo prazo. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202010706,2.02.01.07.06,"Certificados de Recebíveis Imobiliários(CRI) – Longo Prazo","Contas que registram o valor de Certificados de Recebíveis Imobiliários a pagar no longo prazo. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202010707,2.02.01.07.07,"Certificados de Recebíveis do Agronegócio(CRA) – Longo Prazo","Contas que registram o valor de Certificados de Recebíveis do Agronegócio a pagar no longo prazo. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202010725,2.02.01.07.25,"Outros Títulos de Dívida a Pagar - – Pelo Custo Amortizado - Longo Prazo","Contas que registram outros títulos emitidos para a captação de recursos financeiros, não classificados em contas mais específicas, mensurados pelo custo amortizado. Havendo contas societárias específicas para juros e encargos decorrentes, também devem ser classificados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202010727,2.02.01.07.27,"(-) Custos a Amortizar - Títulos de Dívida - Longo Prazo","Contas que registram o valor do custo a amortizar a longo prazo dos títulos de dívidas.",liability_non_current,,l10n_br_account_chart_template
account_template_202010728,2.02.01.07.28,"Outros Títulos de Dívida a Pagar – Pelo Pelo Valor Justo(VJPR) - – Longo Prazo","Contas que registram outros títulos emitidos para a captação de recursos financeiros, não classificados em contas mais específicas, mensurados pelo valor justo reconhecido diretamente no resultado.",liability_non_current,,l10n_br_account_chart_template
account_template_202010729,2.02.01.07.29,"(-) Deságio a Apropriar - Debêntures - Longo Prazo","Contas que registram o valor do deságio aplicado a títulos de dívida.",liability_non_current,,l10n_br_account_chart_template
account_template_202010770,2.02.01.07.70,"Subconta - Ajuste a Valor Justo – Títulos de Dívida a Pagar - – Longo Prazo","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre os passivos financeiros, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da baixa ou liquidação do passivo(arts. 44 e 48, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",liability_non_current,,l10n_br_account_chart_template
account_template_202010901,2.02.01.09.01,"Provisões de Natureza Trabalhista - Longo Prazo","Contas que registram o valor da provisão de natureza trabalhista.",liability_non_current,,l10n_br_account_chart_template
account_template_202010902,2.02.01.09.02,"Provisões de Natureza Tributária - Longo Prazo","Contas que registram o valor da provisão de natureza tributária.",liability_non_current,,l10n_br_account_chart_template
account_template_202010903,2.02.01.09.03,"Provisões de Natureza Cível - Longo Prazo","Contas que registram o valor da provisão de natureza cível.",liability_non_current,,l10n_br_account_chart_template
account_template_202010928,2.02.01.09.28,"Outras Provisões - Longo Prazo","Contas que registram o valor das provisões de longo prazo não classificáveis em contas específicas nesse plano de contas.",liability_non_current,,l10n_br_account_chart_template
account_template_202011001,2.02.01.10.01,"IRRF a Recolher - Longo Prazo","Contas que registram o valor referentes ao IRRF a Recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011002,2.02.01.10.02,"IPI a Recolher - Longo Prazo","Contas que registram o valor referentes ao IPI a Recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011003,2.02.01.10.03,"ICMS a Recolher - Longo Prazo","Contas que registram o valor referente ao ICMS a Recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011004,2.02.01.10.04,"PIS a Recolher - Longo Prazo","Contas que registram o valor do PIS a recolher após após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011005,2.02.01.10.05,"COFINS a Recolher - Longo Prazo","Contas que registram o valor da COFINS a recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011006,2.02.01.10.06,"IOF a Recolher - Longo Prazo","Contas que registram o valor do IOF a recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011007,2.02.01.10.07,"CIDE a Recolher - Longo Prazo","Contas que registram o valor da CIDE a recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011008,2.02.01.10.08,"Tributos Municipais a Recolher - Longo Prazo","Contas que registram o valor dos tributos municipais a Recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011009,2.02.01.10.09,"Parcelamentos Especiais a Recolher - Tributos Federais – Longo Prazo","Contas que registram o valor dos saldos de parcelamentos especiais e ordinários de tributos federais a Recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011010,2.02.01.10.10,"Parcelamentos Especiais a Recolher - Tributos Estaduais e Municipais – Longo Prazo","Contas que registram o valor dos saldos de parcelamentos especiais e ordinários de tributos estaduais e municipais a recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011011,2.02.01.10.11,"Contribuição a Recolher - Desoneração da Folha de Pagamento - Longo Prazo","Contas que registram o valor referente à contribuição sobre o faturamento em substituição ao INSS sobre a folha. Informar o saldo a recolher após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011028,2.02.01.10.28,"Outros Tributos a Recolher - Longo Prazo","Contas que registram o valor dos tributos a recolher não classificáveis em contas específicas após o final do período de apuração seguinte (longo prazo).",liability_non_current,,l10n_br_account_chart_template
account_template_202011101,2.02.01.11.01,"Mútuos - Partes Não Relacionadas – No País - Longo Prazo","Contas que registram os empréstimos tomados de partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, situadas no país. Juros e encargos decorrentes também devem ser registrados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202011102,2.02.01.11.02,"Mútuos - Partes Não Relacionadas - No Exterior - Longo Prazo","Contas que registram os empréstimos tomados de partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, situadas no exterior. Juros e encargos decorrentes também devem ser registrados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202011103,2.02.01.11.03,"Mútuos – Partes Relacionadas – No País - Longo Prazo","Contas que registram os empréstimos tomados de partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, situadas no país. Juros e encargos decorrentes também devem ser registrados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202011104,2.02.01.11.04,"Mútuos - Partes Relacionadas – No Exterior - Longo Prazo","Contas que registram os empréstimos tomados de partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12, situadas no exterior. Juros e encargos decorrentes também devem ser registrados nesta conta.",liability_non_current,,l10n_br_account_chart_template
account_template_202011110,2.02.01.11.10,"Passivo Contingente Assumido em Combinação de Negócios - Longo Prazo","Contas que registram o passivo contingente de longo prazo assumido em uma combinação de negócios.",liability_non_current,,l10n_br_account_chart_template
account_template_202011113,2.02.01.11.13,"Adiantamento para Futuro Aumento de Capital - Passivo - Longo Prazo","Contas que registram o valor dos recursos recebidos pela empresa de seus acionistas ou quotistas destinados a serem utilizados para aumento de capital.",liability_non_current,,l10n_br_account_chart_template
account_template_202011115,2.02.01.11.15,"Conta de Controle de Custo Contratado - Longo Prazo","Contas que registram o controle de custo contratado de longo prazo, utilizadas pelas pessoas jurídicas da atividade imobiliária.",liability_non_current,,l10n_br_account_chart_template
account_template_202011116,2.02.01.11.16,"Conta de Controle de Custo Orçado - Longo Prazo","Contas que registram o controle de custo orçado de longo prazo, utilizadas pelas pessoas jurídicas da atividade imobiliária.",liability_non_current,,l10n_br_account_chart_template
account_template_202011122,2.02.01.11.22,"Direitos Creditórios a Pagar – Longo Prazo","Contas que registram os direitos creditórios a pagar no longo prazo, utilizadas por pessoa jurídica que exerça ativididade de securitização.",liability_non_current,,l10n_br_account_chart_template
account_template_202011128,2.02.01.11.28,"Outras Obrigações - Longo Prazo","Contas que registram outras obrigações na classificadas em contas mais específicas.",liability_non_current,,l10n_br_account_chart_template
account_template_202011150,2.02.01.11.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) Outras Obrigações - Longo Prazo","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados em outras obrigações. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Em sendo as contrapartidas registradas em subcontas vinculadas a ativos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_non_current,,l10n_br_account_chart_template
account_template_202011160,2.02.01.11.60,"CPC 47 - Passivos de Contrato - Longo Prazo","Contas que registram os efeitos no passivo não circulante decorrentes da adoção do Pronunciamento Técnico CPC 47 - Receita de Contrato com Cliente.",liability_non_current,,l10n_br_account_chart_template
account_template_202011170,2.02.01.11.70,"Subconta - Ajuste a Valor Justo – Outras Obrigações - Longo Prazo","Contas que registram os ajustes a valor justo positivos ou negativos efetuados sobre outras obrigações, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da baixa ou liquidação do passivo(arts. 44 e 48, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",liability_non_current,,l10n_br_account_chart_template
account_template_202011190,2.02.01.11.90,"Subconta – Adoção Inicial - Outras Obrigações - Longo Prazo","Contas que registram registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do passivo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida que o passivo for baixado ou liquidado (arts. 165 e 168, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o passivo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",liability_non_current,,l10n_br_account_chart_template
account_template_202012101,2.02.01.21.01,"Receitas Diferidas","Contas que registram o valor das receitas faturadas e recebidas antecipadamente decorrente da venda de bens e serviços, cuja a execução e entrega ocorrem após o término do ano-calendário subsequente. Também se consideram como receitas diferidas os juros e demais receitas financeiras recebidas antecipadamente em transações financeiras.",liability_non_current,,l10n_br_account_chart_template
account_template_202012102,2.02.01.21.02,"(-) Custos Correspondentes às Receitas Diferidas","Contas que registram o valor dos custos e despesas de exercícios futuros referentes às receitas diferidas.",liability_non_current,,l10n_br_account_chart_template
account_template_202012103,2.02.01.21.03,"Subvenção Governamental a Apropriar","Contas que registram o valor das doações e subvenções governamentais, enquanto não transferidas para o resultado do exercício.",liability_non_current,,l10n_br_account_chart_template
account_template_202012150,2.02.01.21.50,"(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Receita Diferida - Longo Prazo","Contas que registram os valores dos juros a serem apropriados como despesa financeira decorrente do ajuste a valor presente efetuados em outras obrigações. Referidos valores serão apropriados ao resultado pelo regime de competência e adicionados ao Lucro Real(art. 38, §2º, Instrução Normativa SRF nº 1.515/2014). Em sendo as contrapartidas registradas em subcontas vinculadas a ativos, poderão ser excluídas no período em que os ativos forem realizados, nos termos do art. 37, Instrução Normativa SRF nº 1.515/2014.",liability_non_current,,l10n_br_account_chart_template
account_template_202012170,2.02.01.21.70,"Subconta - Ajuste a Valor Justo – Receita Diferida - Longo Prazo","Contas que registram registram os ajustes a valor justo positivos ou negativos efetuados sobre receitas diferidas, inclusive decorrentes apenas de sua mensuração inicial. Referidos valores deverão ser computados na apuração Lucro Real quando da baixa ou liquidação do passivo(arts. 44 e 48, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 5/6). Apenas no caso da conta contábil que registra o ativo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 33, §3º, Instrução Normativa SRF nº 1.515/2014)",liability_non_current,,l10n_br_account_chart_template
account_template_202012190,2.02.01.21.90,"Subconta – Adoção Inicial - Receita Diferida - Longo Prazo","Contas que registram registram as diferenças positivas ou negativas, na data da adoção dos efeitos da Lei nº 12.973/2014, entre o valor do passivo mensurado de acordo as disposições da Lei nº 6.404/1976 e o valor mensurado pelos métodos e critérios vigentes em 31/12/2007(arts. 66/67, Lei nº 12.973/2014). Referidos valores deverão ser computados na apuração Lucro Real à medida que o passivo for baixado ou liquidado (arts. 165 e 168, Instrução Normativa SRF nº 1.515/2014, cujos detalhes sobre a contabilização estão descritos em seu Anexo I, exemplos 1/2). Conforme disciplinado art. 169, § 9ª, Instrução Normativa SRF nº 1.515/2014, cada subconta de adoção inicial deve registrar individualmente a diferença de valor identificada em cada ativo. Apenas no caso da conta contábil que registra o passivo consolidar vários itens de mesma natureza, pode-se utilizar uma mesma subconta coletiva, desde que demonstre-os em razão auxiliar vinculado à ECD(art. 169, §6º, Instrução Normativa SRF nº 1.515/2014)",liability_non_current,,l10n_br_account_chart_template
account_template_203010101,2.03.01.01.01,"Capital Subscrito de Domiciliados e Residentes no País","Contas que registram o valor do capital subscrito de domiciliados no País.",equity,,l10n_br_account_chart_template
account_template_203010121,2.03.01.01.21,"(-) Capital a Integralizar de Domiciliados e Residentes no País","Contas que registram o valor do capital social subscrito de domiciliados no país que não tenha sido integralizado.",equity,,l10n_br_account_chart_template
account_template_203010201,2.03.01.02.01,"Capital Subscrito de Domiciliados e Residentes no Exterior","Contas que registram o valor do capital subscrito de domiciliados no exterior.",equity,,l10n_br_account_chart_template
account_template_203010210,2.03.01.02.10,"(-) Capital a Integralizar de Domiciliados e Residentes no Exterior","Contas que registram o valor do capital social subscrito de domiciliados no exterior que não tenha sido integralizado.",equity,,l10n_br_account_chart_template
account_template_203011001,2.03.01.10.01,"(-) Gastos com Emissão de Ações","Contas que registram os gastos com captação de recursos por emissão de ações ou outros valores mobiliários, nos termos do CPC 08(R1). Referidos valores poderão ser excluídos do Lucro Real quando incorridos(art.77, Instrução Normativa SRF nº 1.515/2014)",equity,,l10n_br_account_chart_template
account_template_203020101,2.03.02.01.01,"Ágio na Emissão de Ações","Contas que registram o valor do ágio apurado na emissão de ações.",equity,,l10n_br_account_chart_template
account_template_203020102,2.03.02.01.02,"Reserva Especial de Ágio na Incorporação","Contas que registram a Reserva Especial de Ágio estabelecida no art. 6º, Instrução CVM nº 319/1999.",equity,,l10n_br_account_chart_template
account_template_203020103,2.03.02.01.03,"Alienação de Partes Beneficiárias e Bônus de Subscrição","Contas que registram o valor da emissão e do resgate ou conversão em ações de partes beneficiárias e bônus de subscrição.",equity,,l10n_br_account_chart_template
account_template_203020111,2.03.02.01.11,"Doações e Subvenções para Investimentos (Reserva constituída até 31/12/2007)","Contas que registram o valor das doações e subvenções para investimento, até 31.12.2007.",equity,,l10n_br_account_chart_template
account_template_203020112,2.03.02.01.12,"Prêmio Recebido na Emissão de Debêntures (Reserva constituída até 31/12/2007)","Contas que registram o valor dos prêmios na emissão de debêntures, até 31.12.2007.",equity,,l10n_br_account_chart_template
account_template_203020199,2.03.02.01.99,"Outras Reservas de Capital","Contas que registram o valor das reservas de capital não classificadas em contas específicas nesse plano de contas.",equity,,l10n_br_account_chart_template
account_template_203020201,2.03.02.02.01,"Reserva de Reavaliação","Contas que registram o valor da reserva de reavaliação realizada sobre ativos próprios, ainda não realizada e não estornada até final de 2008, nos termos do art. 6º, Lei nº 11.638/2007. Conforme disciplinado no item 38, CPC 13, sua constituição não é mais permitida.",equity,,l10n_br_account_chart_template
account_template_203020202,2.03.02.02.02,"Reserva de Reavaliação Reflexa","Contas que registram o valor da reserva de reavaliação realizada sobre ativos de coligadas e controladas, ainda não realizada e não estornada até final de 2008, nos termos do art. 6º, Lei nº 11.638/2007. Conforme disciplinado no item 38, CPC 13, sua constituição não é mais permitida.",equity,,l10n_br_account_chart_template
account_template_203020301,2.03.02.03.01,"Reserva Legal","Contas que registram o valor da reserva legal constituída pela destinação de lucros da empresa.",equity,,l10n_br_account_chart_template
account_template_203020302,2.03.02.03.02,"Reserva Estatutária","Contas que registram o valor da reserva estatutária constituída pela destinação de lucros da empresa.",equity,,l10n_br_account_chart_template
account_template_203020303,2.03.02.03.03,"Reserva para Contingência","Contas que registram o valor da reserva para contingência constituída pela destinação de lucros da empresa.",equity,,l10n_br_account_chart_template
account_template_203020304,2.03.02.03.04,"Reserva de Incentivos Fiscais","Contas que registram a Reserva de Incetivos Fiscais criada pelo art. 195-A, Lei nº 6.404/1976. O valor das doações e subvenções governamentais para investimento, a partir de 01.01.2008, deve receber esta destinação.",equity,,l10n_br_account_chart_template
account_template_203020305,2.03.02.03.05,"Reserva de Lucros para Expansão","Contas que registram o valor da reserva de lucros para expansão constituída pela destinação de lucros da empresa.",equity,,l10n_br_account_chart_template
account_template_203020306,2.03.02.03.06,"Reserva de Lucros a Realizar","Contas que registram o valor da reserva de lucros a realizar constituída pela destinação de lucros da empresa.",equity,,l10n_br_account_chart_template
account_template_203020307,2.03.02.03.07,"Reserva Especial para Dividendo Obrigatório não Distribuído","Contas que registram o valor da reserva especial para dividendo obrigatório não distribuído constituída pela destinação de lucros da empresa.",equity,,l10n_br_account_chart_template
account_template_203020308,2.03.02.03.08,"Reserva de Prêmio na Emissão de Debêntures","Contas que registram a Reserva de Prêmio na Emissão de Debêntures prevista no art. 19, inc. III, Lei 11.941/2009.",equity,,l10n_br_account_chart_template
account_template_203020309,2.03.02.03.09,"Reserva para Aumento de Capital (Lei nº 9.249/1995, art. 9º, § 9º)","Contas que registram o valor da reserva constituída em 1996 com o montante dos juros sobre o capital próprio deduzidos como despesa financeira, mas mantidos no patrimônio da empresa.",equity,,l10n_br_account_chart_template
account_template_203020399,2.03.02.03.99,"Outras Reservas de Lucros","Contas que registram o valor das demais reservas de lucros não classificadas em contas mais específicas.",equity,,l10n_br_account_chart_template
account_template_203030101,2.03.03.01.01,"Contrapartidas Ajustes Ativos Financeiros Disponíveis para Venda","Contas que registram o valor das contrapartidas de aumentos ou diminuições de valor atribuídos a ativos financeiros disponíveis para venda, em decorrência da sua avaliação a valor justo. Regra geral, estes ganhos ou perdas devem ser reconhecidos no resultado do período em que foram desreconhecidos, conforme item 55, CPC 38.",equity,,l10n_br_account_chart_template
account_template_203030102,2.03.03.01.02,"Contrapartidas Ajustes em Operações de Hedge","Contas que registram o valor das contrapartidas de aumentos ou diminuições de valor atribuídos instrumentos/itens em operações de hedge, em decorrência da sua avaliação a valor justo. Regra geral, as parcelas dos ganhos ou perdas do hedge eficaz de fluxo de caixa devem receber esta classificação, até que o fluxo protegido afete o resultado, quando estes valores devem ser reclassificados para o resultado do período, conforme itens 95/100, CPC 38.",equity,,l10n_br_account_chart_template
account_template_203030103,2.03.03.01.03,"Contrapartida da Diferença Positiva de Ativo Imobilizado Transferido para Propriedades para Investimento","Contas que registram a contrapartida de diferença positiva de ativo imobilizado transferido para propriedades para investimento quando do seu reconhecimento inicial, nos termos do item 62(b), CPC 28.",equity,,l10n_br_account_chart_template
account_template_203030104,2.03.03.01.04,"Contrapartida de Ajustes dos Planos de Benefícios a Empregados","Contas que registram os ajustes referentes aos planos de benefícios a empregados, nos termos do item 57(d), CPC 33.",equity,,l10n_br_account_chart_template
account_template_203030105,2.03.03.01.05,"Contrapartida de Ajustes do Ativo Imobilizado e de Propriedades para Investimento - Adoção Inicial CPC","Contas que registram contrapartidas dos ajustes, positivos e negativos, ao Imobilizado e Propriedades para Investimento, na adoção inicial dos CPC 27 e CPC 28. Decorrem do custo atribuído, ""deemed cost"", definido no ICPC 10. Conforme item 26 do ICPC 10, na medida em que os ativos forem realizados, correspondentes saldos devem ser transferidos para a conta Lucros ou Prejuízos Acumulados.",equity,,l10n_br_account_chart_template
account_template_203030106,2.03.03.01.06,"Ajustes Acumulados de Conversão","Contas que registram os ajustes acumulados de conversão cambial definidos no CPC 02(R2).",equity,,l10n_br_account_chart_template
account_template_203030130,2.03.03.01.30,"(-) Ajustes de Avaliação Patrimonial Negativos","Contas que registram os ajustes de avaliação patrimonial negativos não classificados em contas específicas.",equity,,l10n_br_account_chart_template
account_template_203030190,2.03.03.01.90,"Outros Resultados Abrangentes - Ajustes de Avaliação Patrimonial","Contas que registram os ajustes de avaliação patrimonial relativos a outros resultados abrangentes.",equity,,l10n_br_account_chart_template
account_template_203030201,2.03.03.02.01,"Contrapartidas Ajustes Ativos Financeiros Disponíveis para Venda - Reflexa","Contas que registram na investidora os ajustes de resultados abrangentes no Patrimônio Líquida da pessoa jurídica investida, classificados na conta 2.03.03.01.01",equity,,l10n_br_account_chart_template
account_template_203030202,2.03.03.02.02,"Contrapartidas Ajustes em Operações de Hedge - Reflexa","Contas que registram na investidora os ajustes de resultados abrangentes no Patrimônio Líquida da pessoa jurídica investida, classificados na conta 2.03.03.01.02",equity,,l10n_br_account_chart_template
account_template_203030203,2.03.03.02.03,"Contrapartida Diferença Positiva de Ativo Imobilizado Transferido para Propriedades para Investimento - Reflexa","Contas que registram na investidora os ajustes de resultados abrangentes no Patrimônio Líquida da pessoa jurídica investida, classificados na conta 2.03.03.01.03",equity,,l10n_br_account_chart_template
account_template_203030204,2.03.03.02.04,"Contrapartida Ajustes Planos de Benefícios a Empregados - Reflexa","Contas que registram registram na investidora os ajustes de resultados abrangentes no Patrimônio Líquida da pessoa jurídica investida, classificados na conta 2.03.03.01.04",equity,,l10n_br_account_chart_template
account_template_203030205,2.03.03.02.05,"Contrapartida Ajustes Imobilizado e Propriedades para Investimento - Adoção Inicial CPC - Reflexa","Contas que registram a contrapartida de ajustes do imobilizado e propriedades para investimento da pessoa jurídica investida na adoção inicial dos CPC.",equity,,l10n_br_account_chart_template
account_template_203030206,2.03.03.02.06,"Ajustes Acumulados de Conversão - Reflexa","Contas que registram registram na investidora os ajustes de resultados abrangentes no Patrimônio Líquida da pessoa jurídica investida, classificados na conta 2.03.03.01.06",equity,,l10n_br_account_chart_template
account_template_203030230,2.03.03.02.30,"(-) Ajustes de Avaliação Patrimonial Negativos - Reflexa","Contas que registram registram na investidora os ajustes de resultados abrangentes no Patrimônio Líquida da pessoa jurídica investida, classificados na conta 2.03.03.01.30",equity,,l10n_br_account_chart_template
account_template_203030290,2.03.03.02.90,"Outros Resultados Abrangentes - Ajustes de Avaliação Patrimonial - Reflexa","Contas que registram na investidora os ajustes de resultados abrangentes no Patrimônio Líquida da pessoa jurídica investida, classificados na conta 2.03.03.01.90",equity,,l10n_br_account_chart_template
account_template_203040101,2.03.04.01.01,"Lucros Acumulados e/ou Saldo à Disposição da Assembleia","Contas que registram o valor dos lucros acumulados ou do saldo à disposição da assembleia.",equity,,l10n_br_account_chart_template
account_template_203040105,2.03.04.01.05,"Contraprestação Contingente - Combinação de Negócios - Patrimônio Líquido","Contas que registram a contraprestação contingente passivas reconhecida pelo adquirente em uma combinação de negócios, nos termos do item 40, CPC15. Em termos gerais, constitui cláusula assumida pelo adquirente de transferir ativos ou participações societárias adicionais , aos ex-proprietários da adquirida, se certas condições específicas venham a ocorrer. Para fins de gerar efeito sobre tratamento fiscal das parcelas integrantes do custo de aquisição de participação societária, deve-se observar os arts. 110/111, Instrução Normativa SRF Nº 1.515/2014 .",equity,,l10n_br_account_chart_template
account_template_203040110,2.03.04.01.10,"Ajustes de Exercícios Anteriores","Contas que registram os ajustes de exercícios anteriores.",equity,,l10n_br_account_chart_template
account_template_203040111,2.03.04.01.11,"(-) Prejuízos Acumulados","Contas que registram o valor aos prejuízos acumulados.",equity,,l10n_br_account_chart_template
account_template_203040112,2.03.04.01.12,"(-) Ações em Tesouraria","Contas que registram o valor das aquisições de ações da própria empresa.",equity,,l10n_br_account_chart_template
account_template_203040115,2.03.04.01.15,"(-) Transações de Capital","Contas que registram as transações de capital efetuadas com os sócios da pessoa jurídica. Nos termos do itens 64/70, ICPC 09(R2), as variações de porcentagem de participação em controladas devem receber esta classificação, inclusive quanto a eventual ágio apurado na transação.",equity,,l10n_br_account_chart_template
account_template_203040190,2.03.04.01.90,"Contas do Patrimônio Líquido Não Classificadas","Contas que registram o valor correspondentes a outras contas de Patrimônio Líquido não classificadas em contas específicas.",equity,,l10n_br_account_chart_template
account_template_30101010101,3.01.01.01.01.01,"Receita de Exportação Direta de Mercadorias e Produtos","Contas que registram o valor da receita auferida em decorrência da exportação direta de mercadorias e produtos.",income,,l10n_br_account_chart_template
account_template_30101010102,3.01.01.01.01.02,"Receita de Vendas de Mercadorias e Produtos a Comercial Exportadora com Fim Específico de Exportação","Contas que registram o valor da receita auferida em decorrência da venda de mercadorias e produtos a empresa comercial exportadora, com fim específico de exportação.",income,,l10n_br_account_chart_template
account_template_30101010103,3.01.01.01.01.03,"Receita de Exportação de Serviços","Contas que registram o valor da receita auferida em decorrência da exportação direta de serviços.",income,,l10n_br_account_chart_template
account_template_30101010104,3.01.01.01.01.04,"Receita da Venda de Produtos de Fabricação Própria no Mercado Interno","Contas que registram a receita auferida no mercado interno correspondente à venda de produtos de fabricação própria e as receitas auferidas na industrialização por encomenda ou por conta e ordem de terceiros. (Não se incluem o valor correspondente ao Imposto sobre Produtos Industrializados (IPI) cobrado destacadamente do comprador ou contratante, uma vez que o vendedor é mero depositário e este imposto não integra o preço de venda da mercadoria, e, também, o valor correspondente ao ICMS cobrado na condição de substituto).",income,,l10n_br_account_chart_template
account_template_30101010105,3.01.01.01.01.05,"Receita da Revenda de Mercadorias no Mercado Interno","Contas que registram o valor da receita auferida no mercado interno, correspondente à revenda de mercadorias e o resultado auferido nas operações de conta alheia.",income,,l10n_br_account_chart_template
account_template_30101010106,3.01.01.01.01.06,"Receita da Prestação de Serviços no Mercado Interno","Contas que registram a receita decorrente dos serviços prestados no mercado interno.",income,,l10n_br_account_chart_template
account_template_30101010107,3.01.01.01.01.07,"Receita da Venda de Unidades Imobiliárias","Montante das receitas das unidades imobiliárias vendidas, apropriadas ao resultado, inclusive os custos recuperados de períodos de apuração anteriores.",income,,l10n_br_account_chart_template
account_template_30101010108,3.01.01.01.01.08,"Receita da Locação de Bens Móveis e Imóveis","Contas que registram a receita decorrente da locação de bens móveis e imóveis.",income,,l10n_br_account_chart_template
account_template_30101010120,3.01.01.01.01.20,"Receita de Contrato de Construção","Contas que registram a receita decorrente de contratos de construção – CPC 17",income,,l10n_br_account_chart_template
account_template_30101010125,3.01.01.01.01.25,"Receita de Direito de Exploração Serviço Público ","Contas que registram a receita decorrente de direitos de exploração de serviços públicos – ICPC 01",income,,l10n_br_account_chart_template
account_template_30101010130,3.01.01.01.01.30,"Receita de Securitização de Créditos","Contas que registram a receita decorrente de operações realizadas por securitizadoras.",income,,l10n_br_account_chart_template
account_template_30101010198,3.01.01.01.01.98,"Outras Receitas da Atividade Geral","Outras contas que registrem valores das demais receitas auferida em decorrência da atividade fim da companhia, esporádica ou recorrentes não especificadas nas demais contas de receita.",income,,l10n_br_account_chart_template
account_template_30101010201,3.01.01.01.02.01,"(-) Vendas Canceladas e Devoluções de Vendas","Contas que registram o valor que correspondam as vendas canceladas e a devoluções de vendas.",expense,,l10n_br_account_chart_template
account_template_30101010202,3.01.01.01.02.02,"(-) Descontos Incondicionais e Abatimentos","Contas que registram o valor que corresponde a descontos incondicionais e abatimentos concedidos.",expense,,l10n_br_account_chart_template
account_template_30101010203,3.01.01.01.02.03,"(-) ICMS","Contas que registram o total do Imposto Sobre Operações Relativas à Circulação de Mercadorias e Sobre Prestação de Serviços de Transporte Interestadual e Intermunicipal e de Comunicação (ICMS) calculado sobre as receitas das vendas e de serviços.  Informar o resultado da aplicação das alíquotas sobre as respectivas receitas, e não o montante recolhido, durante o período de apuração, pela pessoa jurídica.  O valor referente ao ICMS pago como substituto não deve ser incluído nesta conta.",expense,,l10n_br_account_chart_template
account_template_30101010204,3.01.01.01.02.04,"(-) COFINS Sobre Receita Bruta","Contas que registram o valor total da COFINS apurada sobre a receita de vendas em consonância com a legislação vigente à época da ocorrência dos fatos geradores. O valor informado deve ser apurado de forma centralizada pelo estabelecimento matriz, quando a pessoa jurídica possuir mais de um estabelecimento (Lei nº 9.779, de 1999, art. 15, III).  Não incluir a COFINS incidente sobre as demais receitas operacionais, que deverá ser informada em conta distinta.",expense,,l10n_br_account_chart_template
account_template_30101010205,3.01.01.01.02.05,"(-) PIS/PASEP Sobre Receita Bruta","Contas que registram o valor total das contribuições para o PIS/PASEP apurado sobre a receita de vendas em consonância com a legislação vigente à época da ocorrência dos fatos geradores. O valor informado deve ser apurado de forma centralizada pelo estabelecimento matriz, quando a pessoa jurídica possuir mais de um estabelecimento (Lei nº 9.779, de 1999, art. 15, III). Não incluir a COFINS incidente sobre as demais receitas operacionais, que deverá ser informada em conta distinta.",expense,,l10n_br_account_chart_template
account_template_30101010206,3.01.01.01.02.06,"(-) ISS","Contas que registram o Imposto sobre Serviço de qualquer Natureza (ISS) relativo às receitas de serviços, conforme legislação específica.",expense,,l10n_br_account_chart_template
account_template_30101010209,3.01.01.01.02.09,"(-) Demais Impostos e Contribuições Incidentes sobre Vendas e Serviços","Contas que registrem os demais impostos e contribuições incidentes sobre as receitas das vendas de que tratam as contas integrantes do grupo RECEITA BRUTA, que guardem proporcionalidade com o preço e sejam considerados redutores das receitas de vendas.",expense,,l10n_br_account_chart_template
account_template_30101010210,3.01.01.01.02.10,"(-) Ajuste a Valor Presente sobre Receita Bruta","Contas que registram os expurgos dos efeitos do ajuste a valor presente sobre a receita bruta.",expense,,l10n_br_account_chart_template
account_template_30101010260,3.01.01.01.02.60,"(-) CPC 47 - Modificações Contratuais","Contas que registram procedimentos contábeis decorrentes da alteração ou adoção de novos métodos ou critérios contábeis relacionadas a modificações contratuais (item 21 do CPC 47)",expense,,l10n_br_account_chart_template
account_template_30101010262,3.01.01.01.02.62,"(-) CPC 47 - Reconhecimento de Passivos de Contrato - Garantias","Contas que registram procedimentos contábeis decorrentes da alteração ou adoção de novos métodos ou critérios contábeis relacionadas ao reconhecimento de passivos em razão de obrigações contratuais relativas a garantias, exceto as contratadas com empresas de seguros e as contabilizadas como provisões (itens B30, B31 e B32 do CPC 47).",expense,,l10n_br_account_chart_template
account_template_30101010264,3.01.01.01.02.64,"(-) CPC 47 - Reconhecimento de Passivos de Contrato - Direitos não Exercidos","Contas que registram procedimentos contábeis decorrentes da alteração ou adoção de novos métodos ou critérios contábeis relacionadas ao reconhecimento de passivos em razão de obrigações contratuais relativas a direitos não exercidos (item B46 do CPC 47).",expense,,l10n_br_account_chart_template
account_template_30101010266,3.01.01.01.02.66,"(-) CPC 47 - Reconhecimento de Passivos de Contrato - Serviços de Custódia - Vendas para Entrega Futura","Contas que registram procedimentos contábeis decorrentes da alteração ou adoção de novos métodos ou critérios contábeis relacionadas ao reconhecimento de passivos em razão de obrigações contratuais relativas a serviços de custódia, na hipótese de vendas para entrega futura (item B82 do CPC 47).",expense,,l10n_br_account_chart_template
account_template_30101010268,3.01.01.01.02.68,"(-) CPC 47 - Preço de Transação - Contraprestações Variáveis","Contas que registram procedimentos contábeis decorrentes da alteração ou adoção de novos métodos ou critérios contábeis relacionadas à determinação do preço de transação (itens 46, 47 e 48 do CPC 47) em razão do reconhecimento de contraprestações variáveis (itens 50 e 56 do CPC 47), nas hipóteses não previstas nos itens 21, B30, B31, B32, B46 e B82 do CPC 47.",expense,,l10n_br_account_chart_template
account_template_30101010270,3.01.01.01.02.70,"(-) CPC 47 - Preço de Transação - Reavaliações de Contraprestação Variável","Contas que registram procedimentos contábeis decorrentes da alteração ou adoção de novos métodos ou critérios contábeis relacionadas à determinação do preço de transação (itens 46, 47 e 48 do CPC 47) em razão do reconhecimento de reavaliações da contraprestação variável (item 59 do CPC 47).",expense,,l10n_br_account_chart_template
account_template_30101010272,3.01.01.01.02.72,"(-) CPC 47 - Preço de Transação - Contraprestações Pagas ou a Pagar","Contas que registram procedimentos contábeis decorrentes da alteração ou adoção de novos métodos ou critérios contábeis relacionadas à determinação do preço de transação (itens 46, 47 e 48 do CPC 47) em razão do reconhecimento de contraprestações pagas ou a pagar (itens 70 a 72 do CPC 47).",expense,,l10n_br_account_chart_template
account_template_30101010274,3.01.01.01.02.74,"(-) CPC 47 - Preço de Transação - Obrigações de Desempenho","Contas que registram procedimentos contábeis decorrentes da alteração ou adoção de novos métodos ou critérios contábeis relacionadas à alocação do preço de transação às obrigações de desempenho (itens 73 e 74 do CPC 47), nos casos não previstos nos itens 21, B30, B31, B32, B46 e B82 do CPC 47.2",expense,,l10n_br_account_chart_template
account_template_30101010276,3.01.01.01.02.76,"(-) CPC 47 - Critérios Divergentes da Legislação Tributária - Não Recebimento de Contraprestação","Contas que registram procedimentos contábeis que contemplam métodos ou critérios contábeis que divergem da legislação tributária em relação à aplicação da possibilidade de a entidade não receber a contraprestação a que tem direito na identificação do contrato (item 9.(e) do CPC 47).",expense,,l10n_br_account_chart_template
account_template_30101010278,3.01.01.01.02.78,"(-) CPC 47 - Critérios Divergentes da Legislação Tributária - Passivos de Contrato - Direito à Devolução","Contas que registram procedimentos contábeis que contemplam métodos ou critérios contábeis que divergem da legislação tributária em relação ao reconhecimento de passivos em razão de obrigações contratuais relativas a direito à devolução (itens B21 a B27 do CPC 47).",expense,,l10n_br_account_chart_template
account_template_30101010280,3.01.01.01.02.80,"(-) CPC 47 - Critérios Divergentes da Legislação Tributária - Passivos de Contrato - Direito de Aquisição Opcional","Contas que registram procedimentos contábeis que contemplam métodos ou critérios contábeis que divergem da legislação tributária quanto ao reconhecimento de passivos em razão de obrigações contratuais relativas a direitos de aquisição opcional de bens ou serviços adicionais ou com desconto (item B40 do CPC 47).",expense,,l10n_br_account_chart_template
account_template_30101030101,3.01.01.03.01.01,"(-) Custo dos Produtos de Fabricação Própria Vendidos","Contas que registram o valor dos gastos que compõem o custo total de produção própria após a realização dos estoques. ",expense_direct_cost,,l10n_br_account_chart_template
account_template_30101030102,3.01.01.03.01.02,"(-) Custo das Mercadorias Revendidas","Contas que registram o valor dos gastos que compõem o custo total das mercadorias revendidas após a realização dos estoques. ",expense_direct_cost,,l10n_br_account_chart_template
account_template_30101030103,3.01.01.03.01.03,"(-) Custo dos Serviços Prestados","Contas que registram o valor dos gastos que compõem o custo total da prestação de serviço.",expense_direct_cost,,l10n_br_account_chart_template
account_template_30101030104,3.01.01.03.01.04,"(-) Custo das Unidades Imobiliárias Vendidas","Contas que registram o valor dos gastos que compõem o custo total das unidades imobiliárias vendidas.",expense_direct_cost,,l10n_br_account_chart_template
account_template_30101030110,3.01.01.03.01.10,"(-) Custo dos Bens Arrendados","Contas que registram o valor dos gastos que compõem o custo dos bens (ativos) arrendados.",expense_direct_cost,,l10n_br_account_chart_template
account_template_30101030120,3.01.01.03.01.20,"(-) Custo de Construção","Contas que registram o valor dos custos de contratos de construção",expense_direct_cost,,l10n_br_account_chart_template
account_template_30101030130,3.01.01.03.01.30,"(-) Custo de Operação de Securitização","Contas que registram o valor dos custos derivados da operação de securitização",expense_direct_cost,,l10n_br_account_chart_template
account_template_30101050101,3.01.01.05.01.01,"Variações Cambiais Ativas","Contas que registram os ganhos apurados em razão de variações ativas decorrentes da atualização dos direitos de crédito e obrigações, calculados com base nas variações nas taxas de câmbio.",income_other,,l10n_br_account_chart_template
account_template_30101050102,3.01.01.05.01.02,"Ganhos Auferidos no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório dos ganhos auferidos, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País;
Atenção:
1) Os ganhos auferidos em operações day-trade devem ser informados em conta específica.
2) O valor correspondente às perdas incorridas no mercado de renda variável, exceto day-trade, deve ser informado em conta específica.
3) São consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros as entidades cujo objeto social seja análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM).",income_other,,l10n_br_account_chart_template
account_template_30101050103,3.01.01.05.01.03,"Ganhos em Operações Day-Trade","Contas que registram os ganhos diários auferidos, em cada mês do período de apuração, em operações day-trade.  Considera-se ganho o resultado positivo auferido nas operações citadas acima, realizadas em cada mês, admitida a dedução dos custos e despesas incorridos, necessários à realização das operações.  Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado à vista, no mesmo dia.  Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia.",income_other,,l10n_br_account_chart_template
account_template_30101050104,3.01.01.05.01.04,"Receitas de Juros sobre o Capital Próprio","Contas que registram os juros recebidos, a título de remuneração do capital próprio, em conformidade com o art. 9º da Lei nº 9.249, de 1995. O valor informado deve corresponder ao total dos juros recebidos antes do desconto do imposto de renda na fonte.
O valor do imposto de renda retido na fonte, para as pessoas jurídicas tributadas pelo lucro real, é considerado antecipação do imposto devido no encerramento do período de apuração ou, ainda, pode ser compensado com aquele que for retido, pela beneficiária, por ocasião do pagamento ou crédito de juros a título de remuneração do capital próprio, ao seu titular ou aos seus sócios. ",income_other,,l10n_br_account_chart_template
account_template_30101050105,3.01.01.05.01.05,"Outras Receitas Financeiras","Contas que registram receitas auferidas no período de apuração relativas a juros, descontos, lucro na operação de reporte, prêmio de resgate de títulos ou debêntures e rendimento nominal auferido em aplicações financeiras de renda fixa, não incluídas em linhas específicas. As receitas dessa natureza, derivadas de operações com títulos vencíveis após o encerramento do período de apuração, serão rateadas segundo o regime de competência.",income_other,,l10n_br_account_chart_template
account_template_30101050106,3.01.01.05.01.06,"Resultados Positivos em Participações Societárias Avaliadas pelo Método de Equivalência Patrimonial","Contas que registram o ganho de investimento avaliado pelo método da equivalência patrimonial.",income_other,,l10n_br_account_chart_template
account_template_30101050107,3.01.01.05.01.07,"Resultados Positivos em SCP Avaliadas pelo Método de Equivalência Patrimonial","Esta conta é utilizada pelas pessoas jurídicas que forem sócias ostensivas de sociedades em conta de participação, para a indicação:
a) de lucros derivados de participação em SCP, avaliadas pelo custo de aquisição;
b) dos ganhos por ajustes no valor de participação em SCP, avaliadas pelo método da equivalência patrimonial.
Os lucros recebidos de investimento em SCP, avaliado pelo custo de aquisição, ou a contrapartida do ajuste do investimento ao valor do patrimônio líquido da SCP, no caso de investimento avaliado por esse método, podem ser excluídos na determinação do lucro real dos sócios, pessoas jurídicas, das referidas sociedades (Decreto nº 3.000, de 1999, art. 149).",income_other,,l10n_br_account_chart_template
account_template_30101050108,3.01.01.05.01.08,"Rendimentos e Ganhos de Capital Auferidos no Exterior","Contas que registram os rendimentos e ganhos de capital auferidos no exterior diretamente pela pessoa jurídica domiciliada no Brasil, pelos seus valores antes de descontado o tributo pago no país de origem. Esses valores podem, no caso de apuração trimestral do imposto, ser excluídos na apuração do lucro real do 1º aos 3º trimestres, devendo ser adicionados ao lucro líquido na apuração do lucro real referente ao 4º trimestre.
Atenção: Os ganhos de capital referentes a alienações de bens e direitos do ativo não-circulante, exceto os classificáveis no ativo realizável a longo prazo, situados no exterior devem ser informados em outras receitas.",income_other,,l10n_br_account_chart_template
account_template_30101050109,3.01.01.05.01.09,"Reversão das Perdas Estimadas Decorrentes de Teste de Recuperabilidade (Impairment)"," Contas que registram o ganho decorrente da reversão das perdas estimadas decorrentes da aplicação de teste de recuperabilidade sobre os ativos. ",income_other,,l10n_br_account_chart_template
account_template_30101050110,3.01.01.05.01.10,"Reversão dos Saldos das Provisões","Contas que registram a reversão dos saldos não utilizados das provisões constituídas no balanço do período de apuração imediatamente anterior e ou constituída no próprio período de apuração para fins de apuração do lucro real.",income_other,,l10n_br_account_chart_template
account_template_30101050111,3.01.01.05.01.11,"Prêmios Recebidos na Emissão de Debêntures","Contas que registram valor dos prêmios recebidos na emissão de debêntures, tais como:
1) A pessoa jurídica poderá excluir o valor decorrente de prêmios recebidos na emissão de debêntures, reconhecido no exercício, para fins de apuração do lucro real; caso mantenha em reserva de lucros específica a parcela decorrente de prêmio na emissão de debêntures, apurada até o limite do lucro líquido do exercício;
2) O prêmio na emissão de debêntures será tributado caso seja dada destinação diversa da que está prevista no item 1 acima, inclusive nas hipóteses de:
a) capitalização do valor e posterior restituição de capital aos sócios ou ao titular, mediante redução do capital social, hipótese em que a base para a incidência será o valor restituído, limitado ao valor total das exclusões decorrentes de prêmios na emissão de debêntures;
b) restituição de capital aos sócios ou ao titular, mediante redução do capital social, nos 5 (cinco) anos anteriores à data da emissão das debêntures com o prêmio, com posterior capitalização do valor do prêmio, hipótese em que a base para a incidência será o valor restituído, limitado ao valor total das exclusões decorrentes de prêmios na emissão de debêntures; ou
c) integração à base de cálculo dos dividendos obrigatórios.",income_other,,l10n_br_account_chart_template
account_template_30101050112,3.01.01.05.01.12,"Doações e Subvenções para Custeio ou Operações","Contas que registram as subvenções para custeio ou operações recebidas, inclusive mediante isenção ou redução de impostos concedidas como estímulo à implantação ou expansão de empreendimentos econômicos, e as doações recebidas do Poder Público.",income_other,,l10n_br_account_chart_template
account_template_30101050113,3.01.01.05.01.13,"Doações e Subvenções para Investimentos","Contas que registras as subvenções para investimento recebidas, inclusive mediante isenção ou redução de impostos concedidas como estímulo à implantação ou expansão de empreendimentos econômicos, e as doações recebidas do Poder Público.
Atenção:
1) A pessoa jurídica poderá excluir o valor decorrente de doações ou subvenções governamentais para investimentos, reconhecido no exercício, para fins de apuração do lucro real; caso mantenha em reserva de lucros a que se refere o art. 195-A da Lei nº 6.404, de 1976, a parcela decorrente de doações ou subvenções governamentais, apurada até o limite do lucro líquido do exercício.
2) As doações e subvenções serão tributadas caso seja dada destinação diversa da prevista no item 1, inclusive nas hipóteses de:
a) capitalização do valor e posterior restituição de capital aos sócios ou ao titular, mediante redução do capital social, hipótese em que a base para a incidência será o valor restituído, limitado ao valor total das exclusões decorrentes de doações ou subvenções governamentais para investimentos;
b) restituição de capital aos sócios ou ao titular, mediante redução do capital social, nos 5 (cinco) anos anteriores à data da doação ou da subvenção, com posterior capitalização do valor da doação ou da subvenção, hipótese em que a base para a incidência será o valor restituído, limitado ao valor total das exclusões decorrentes de doações ou de subvenções governamentais para investimentos; ou
c) integração à base de cálculo dos dividendos obrigatórios.
3) Se, no período base em que ocorrer a exclusão, a pessoa jurídica apurar prejuízo contábil ou lucro líquido contábil inferior à parcela decorrente de doações e subvenções governamentais, e neste caso não puder ser constituída como parcela de lucros nos termos do item 1 acima, esta deverá ocorrer nos exercícios subsequentes.",income_other,,l10n_br_account_chart_template
account_template_30101050114,3.01.01.05.01.14,"Receitas de Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram as contrapartidas da realização do grupo Ajustes de Avaliação Patrimonial do PL reconhecidas no resultado do exercício em obediência ao regime de competência.",income_other,,l10n_br_account_chart_template
account_template_30101050115,3.01.01.05.01.15,"Receitas de Reclassificação de Ajustes de Avaliação Patrimonial - Reflexo","Contas que registram as contrapartidas da realização do grupo Ajustes de Avaliação Patrimonial do PL - Reflexo reconhecidas no resultado do exercício em obediência ao regime de competência.",income_other,,l10n_br_account_chart_template
account_template_30101050116,3.01.01.05.01.16,"Receitas Financeiras Decorrentes de Ajustes ao Valor Presente","Contas que registram as contrapartidas de aumentos de ativos sujeitos a Ajuste ao Valor Presente de acordo com o regime de competência.",income_other,,l10n_br_account_chart_template
account_template_30101050117,3.01.01.05.01.17,"Ganho Por Compra Vantajosa em Investimentos","Contas que registram os ganhos auferidos por compra vantajosa nas aquisições de controle de investimento, independente do critério de avaliação.",income_other,,l10n_br_account_chart_template
account_template_30101050118,3.01.01.05.01.18,"Amortização de Menos-Valia","Contas que registram a amortização de menos-valia.",income_other,,l10n_br_account_chart_template
account_template_30101050119,3.01.01.05.01.19,"Receita de Aluguel de Bens Imóveis - Atividade Não Principal","Contas que registram aluguéis de bens por empresa que não tenha por objeto a locação de imóveis.",income_other,,l10n_br_account_chart_template
account_template_30101050120,3.01.01.05.01.20,"Receita de Aluguel de Bens Móveis - Atividade Não Principal","Contas que registram aluguéis de bens por empresa que não tenha por objeto a locação de móveis.",income_other,,l10n_br_account_chart_template
account_template_30101050121,3.01.01.05.01.21,"Créditos Presumidos de IPI","Contas que registram os créditos presumidos do IPI para ressarcimento do valor da Contribuição ao PIS/PASEP e COFINS.",income_other,,l10n_br_account_chart_template
account_template_30101050122,3.01.01.05.01.22,"Créditos Presumidos de PIS/COFINS","Contas que registram o crédito presumido da contribuição para o PIS/PASEP e da COFINS concedido na forma do art. 3º da Lei nº 10.147, de 2000.",income_other,,l10n_br_account_chart_template
account_template_30101050123,3.01.01.05.01.23,"Outros Créditos Fiscais Presumidos","Contas que registram outros créditos fiscais presumidos.",income_other,,l10n_br_account_chart_template
account_template_30101050124,3.01.01.05.01.24,"Multas e Outras Vantagens Recebidas","Contas que registram multas ou vantagens a título de indenização em virtude de rescisão contratual (Lei nº 9.430, de 1996, art. 70, § 3º, II).",income_other,,l10n_br_account_chart_template
account_template_30101050125,3.01.01.05.01.25,"Lucros e Dividendos Derivados de Participações Societárias Avaliadas pelo Custos de Aquisição","Contas que registram os resultados positivos em participações societárias avaliadas pelo custo de aquisição.",income_other,,l10n_br_account_chart_template
account_template_30101050126,3.01.01.05.01.26,"Receitas com Empréstimos de Valores Mobiliários","Contas que registram as receitas com empréstimos de valores mobiliários.",income_other,,l10n_br_account_chart_template
account_template_30101050127,3.01.01.05.01.27,"Rendimentos Auferidos em Operações de Mútuo – Partes Relacionadas","Contas que registram os juros auferidos em operações de mútuo com partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",income_other,,l10n_br_account_chart_template
account_template_30101050128,3.01.01.05.01.28,"Rendimentos Auferidos em Operações de Mútuo – Partes Não Relacionadas","Contas que registram os juros auferidos em operações de mútuo com partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",income_other,,l10n_br_account_chart_template
account_template_30101050129,3.01.01.05.01.29,"Rendimentos Auferidos com Debêntures - Emitente Partes  Relacionadas","Contas que registram os juros auferidos com debêntures emitidas por partes relacionadas.",income_other,,l10n_br_account_chart_template
account_template_30101050130,3.01.01.05.01.30,"Rendimentos Auferidos com Debêntures - Emitente Partes Não Relacionadas","Contas que registram os juros auferidos com debêntures emitidas por partes não relacionadas.",income_other,,l10n_br_account_chart_template
account_template_30101050131,3.01.01.05.01.31,"Rendimentos Auferidos com Títulos Públicos","Contas que registram os juros auferidos em certificados de depósitos bancários (CDB).",income_other,,l10n_br_account_chart_template
account_template_30101050132,3.01.01.05.01.32,"Juros Auferidos com Outros Ativos Financeiros Mensurados Pelo Custo Amortizado","Contas que registram os juros auferidos com outros ativos financeiros mensurados pelo custo amortizado.",income_other,,l10n_br_account_chart_template
account_template_30101050133,3.01.01.05.01.33,"Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  para Negociação - Não Hedge – Valor Justo pelo Resultado (VJPR).","Contas que registram os ganhos decorrentes de ajustes a valor justo de instrumentos financeiros para negociação, exceto hedge, avaliados a valor justo pelo resultado.",income_other,,l10n_br_account_chart_template
account_template_30101050134,3.01.01.05.01.34,"Ganho de Ajustes a Valor Justo  - Instrumentos Financeiros  Disponíveis para Venda - Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram os ganhos decorrentes de ajustes a valor justo de instrumentos financeiros disponíveis para venda no momento da reclassificação dos ajustes de avaliação patrimonial.",income_other,,l10n_br_account_chart_template
account_template_30101050135,3.01.01.05.01.35,"Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  de Hedge  de Valor Justo","Contas que registram os ganhos decorrentes de ajustes a valor justo de instrumentos financeiros de hedge.",income_other,,l10n_br_account_chart_template
account_template_30101050136,3.01.01.05.01.36,"Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  de Hedge -  Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram os ganhos decorrentes de ajustes a valor justo de instrumentos financeiros de hedge no momento da reclassificação dos ajustes de avaliação patrimonial.",income_other,,l10n_br_account_chart_template
account_template_30101050137,3.01.01.05.01.37,"Ganho de Ajustes a Valor Justo - Item Objeto de Hedge de Valor Justo","Contas que registram os ganhos decorrentes de ajustes a valor justo de item objeto de hedge.",income_other,,l10n_br_account_chart_template
account_template_30101050138,3.01.01.05.01.38,"Ganho de Ajustes a Valor Justo - Propriedade para Investimento","Contas que registram os ganhos decorrentes de ajustes a valor justo de propriedades para investimento.",income_other,,l10n_br_account_chart_template
account_template_30101050139,3.01.01.05.01.39,"Ganho de Ajustes a Valor Justo - Ativo Biológico Consumível","Contas que registram os ganhos decorrentes de ajustes a valor justo de ativo biológico consumível.",income_other,,l10n_br_account_chart_template
account_template_30101050140,3.01.01.05.01.40,"Ganho de Ajustes a Valor Justo - Ativo Biológico de Produção","Contas que registram os ganhos decorrentes de ajustes a valor justo de ativo biológico de produção.",income_other,,l10n_br_account_chart_template
account_template_30101050141,3.01.01.05.01.41,"Ganho de Ajustes a Valor Justo - Ativos Não Circulantes Mantidos para Venda","Contas que registram os ganhos decorrentes de ajustes a valor justo de ativos não circulantes mantidos para venda.",income_other,,l10n_br_account_chart_template
account_template_30101050142,3.01.01.05.01.42,"Ganho de Ajustes a Valor Justo - Subscrição de Capital com demais Bens","Contas que registram os ganhos decorrentes de ajustes a valor justo de subscrição de capital com demais bens.",income_other,,l10n_br_account_chart_template
account_template_30101050143,3.01.01.05.01.43,"Ganho de Ajustes a Valor Justo - Subscrição de Capital com Participação Societária","Contas que registram os ganhos decorrentes de ajustes a valor justo de subscrição de capital com participação societária.",income_other,,l10n_br_account_chart_template
account_template_30101050144,3.01.01.05.01.44,"Ganho de Ajustes a Valor Justo - Aquisição de Participação Societária em Estágios","Contas que registram os ganhos decorrentes de ajustes a valor justo de aquisição de participação societária em estágios.",income_other,,l10n_br_account_chart_template
account_template_30101050145,3.01.01.05.01.45,"Ganho de Ajustes a Valor Justo - Decorrente de Permuta de Ativos ou Passivos","Contas que registram os ganhos decorrentes de ajustes a valor justo devido a permuta de ativos ou passivos.",income_other,,l10n_br_account_chart_template
account_template_30101050146,3.01.01.05.01.46,"Ganho de Ajustes a Valor Justo - Outras Operações","Contas que registram os ganhos decorrentes de ajustes a valor justo de outras operações não classificáveis neste plano de contas.",income_other,,l10n_br_account_chart_template
account_template_30101050148,3.01.01.05.01.48,"Cash Discount Gain","",income_other,,l10n_br_account_chart_template
account_template_30101050199,3.01.01.05.01.99,"Outras Receitas Operacionais","Contas que registrem as demais receitas que, por definição legal, sejam consideradas operacionais. tais como recuperações de despesas operacionais de períodos de apuração anteriores, tais como: prêmios de seguros, importâncias levantadas das contas vinculadas do FGTS, ressarcimento de desfalques, roubos e furtos, etc. As recuperações de custos e despesas no decurso do próprio período de apuração devem ser creditadas diretamente às contas de resultado em que foram debitadas.",income_other,,l10n_br_account_chart_template
account_template_30101070101,3.01.01.07.01.01,"(-) Remuneração a Dirigentes e a Conselho de Administração","Contas que registram a despesa incorrida relativa à remuneração mensal e fixa atribuída ao titular de firma individual, aos sócios, diretores e administradores de sociedades, ou aos representantes legais de sociedades estrangeiras, as despesas incorridas com os salários indiretos concedidos pela empresa a administradores, diretores, gerentes e seus assessores (PN Cosit nº 11, de 1992), e o valor referente às remunerações atribuídas aos membros do conselho fiscal ou consultivo.
Atenção:
1) Os valores das gratificações aos dirigentes que estejam ligados à área industrial ou de produção de serviços devem ser informados nas contas de custos, respectivamente;
2) O valor de 13º salário pago a diretor contratado nos termos da Consolidação das Leis do Trabalho (CLT) é dedutível, desde que ele não esteja enquadrado no conceito de sócio, diretor ou administrador estabelecido no PN CST nº 48, de 1972.
3) As gratificações espontâneas devem ser adicionadas ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_30101070102,3.01.01.07.01.02,"(-) Ordenados, Salários, Gratificações e Outras Remunerações a Empregados","Contas que registram as despesas com ordenados, salários, gratificações e outras remunerações a empregados, que não possuam contas específicas no registro L300.
Atenção:
1) As despesas correspondentes a salários, ordenados, gratificações e outras remunerações referentes à área de saúde, tais como assistência médica, odontológica e farmacêutica, devem ser indicadas na conta ""Assistência, médica, odontológica e farmacêutica a empregados"".
2) As despesas com Fundo de Aposentadoria Individual (FAPI), Plano de Poupança e Investimentos (PAIT) e benefícios previdenciários a empregados devem ser indicadas nas contas ""Fundo de Aposentadoria Individual - FAPI"", ""Planos de Poupança e Investimentos - PAIT"" e ""Benefícios Previdenciários a Empregados"", respectivamente.
3) Não deve ser informado nesta linha o valor referente às participações dos empregados no lucro da pessoa jurídica. Esse valor deve ser informado na conta ""Participações de empregados"".
4) O valor das contribuições não compulsórias, destinadas a custear benefícios complementares assemelhados aos da previdência social, instituídos em favor dos empregados e dirigentes da pessoa jurídica, e para os Fundos de Aposentadoria Programada Individual (Fapi) cujo ônus seja da pessoa jurídica, que exceder, no período de apuração, a vinte por cento do total dos salários dos empregados e da remuneração dos dirigentes da empresa, vinculados ao referido plano, deve ser adicionado ao Lucro Real.
5) As demais contribuições não compulsórias, exceto as destinadas a custear seguros e planos de saúde, devem ser adicionados ao Lucro Real"".
",expense,,l10n_br_account_chart_template
account_template_30101070103,3.01.01.07.01.03,"(-) Outros Gastos com Pessoal","Contas que registrem os demais gastos com pessoal não especificados em contas anteriores.",expense,,l10n_br_account_chart_template
account_template_30101070104,3.01.01.07.01.04,"(-) Outros Serviços Prestados por Pessoa Física ou Jurídica","Contas que registram o valor das despesas correspondentes aos serviços prestados por:
1) Pessoa física: que não tenha vínculo empregatício com a pessoa jurídica declarante, tais como: comissões, corretagens, gratificações, honorários, direitos autorais e outras remunerações, inclusive as relativas a empreitadas de obras exclusivamente de trabalho e as decorrentes de fretes e carretos em geral.
2) Pessoa jurídica inclusive cooperativa de trabalho e locação de mão de obra;
Atenção: Somente são dedutíveis as despesas de comissões e corretagens quando, sobre elas, o credor tenha direito líquido e certo (PN CST nº 07, de 28 de janeiro de 1976).",expense,,l10n_br_account_chart_template
account_template_30101070105,3.01.01.07.01.05,"(-) Encargos Sociais - Previdência Social","Contas que registram as contribuições para a Previdência Social, não computadas nos custos (inclusive dos dirigentes - PN CST nº 35, de 31 de agosto de 1981).",expense,,l10n_br_account_chart_template
account_template_30101070106,3.01.01.07.01.06,"(-) Encargos Sociais - FGTS","Contas que registram as contribuições para o FGTS não computadas nos custos (inclusive dos dirigentes - PN CST nº 35, de 31 de agosto de 1981).",expense,,l10n_br_account_chart_template
account_template_30101070107,3.01.01.07.01.07,"(-) Encargos Sociais – Outros","Contas que registram os demais encargos sociais, não computadas nos custos ou nas contas Encargos Sociais -  Previdência Social ou Encargos Sociais -  FGTS.",expense,,l10n_br_account_chart_template
account_template_30101070108,3.01.01.07.01.08,"(-) Doações e Patrocínios de Caráter Cultural e Artístico (Lei no 8.313/1991)","Contas que registram as doações e patrocínios efetuados no período de apuração em favor de projetos culturais previamente aprovados pelo Ministério da Cultura ou pela Agência Nacional do Cinema (Ancine), observada a legislação de concessão dos projetos.
A pessoa jurídica que tiver efetuado doação ou patrocínio a projeto aprovado nos termos dos arts. 25 e 26 da Lei nº 8.313, de 23 de dezembro de 1991, ou nos termos desses dois artigos combinados com o § 6º do art. 39 da Medida Provisória nº 2.228-1, de 6 de agosto de 2001, cujos projetos são produzidos com os recursos de que trata o inciso X desse mesmo art. 39, pode deduzir o valor relativo às doações e/ou patrocínios como despesa operacional.
A pessoa jurídica que tiver efetuado doação ou patrocínio a projeto aprovado nos termos do art.18 da Lei nº 8.313, de 1991, com alterações promovidas pelo art. 1º da Lei nº 9.874, de 23 de novembro de 1999, e pelo art. 53 da MP nº 2.228-1, de 2001, com a redação dada pela Lei nº 10.454, de 2002, ou nos termos desses artigos combinados com o § 6º do art. 39 da Medida Provisória nº 2.228-1, de 6 de agosto de 2001, não pode efetuar qualquer dedução do valor correspondente às doações ou patrocínios como despesa operacional. Esse valor deve ser adicionado ao Lucro Real.
Atenção: Somente podem usufruir os benefícios fiscais referidos nesta linha os incentivadores que obedecerem, para suas doações ou patrocínios, o período definido pelas portarias editadas pelo MinC ou Ancine, publicadas no Diário Oficial da União, para homologação dos projetos beneficiários.",expense,,l10n_br_account_chart_template
account_template_30101070109,3.01.01.07.01.09,"(-) Operações de Aquisição de Vale Cultura (Lei no 12.761/2012, art. 10).","Contas que registram o total do valor despendido no período de apuração a título de aquisição do vale-cultura.
O limite de dedução no percentual de um por cento será considerado isoladamente e não se submeterá a limite conjunto com outras deduções do imposto a título de incentivo. O valor excedente ao limite de dedução não poderá ser deduzido do imposto em períodos de apuração posteriores.
A pessoa jurídica beneficiária:
a) poderá deduzir o valor despendido a título de aquisição do vale-cultura como despesa operacional para fins de apuração do IRPJ; e
b) deverá adicionar o valor deduzido como despesa operacional, para fins de apuração da base de cálculo da CSLL.",expense,,l10n_br_account_chart_template
account_template_30101070110,3.01.01.07.01.10,"(-) Doações a Instituições de Ensino e Pesquisa (Lei nº 9.249/1995, art.13, § 2º)","Contas que registram as doações efetuadas às instituições de ensino e pesquisa cuja criação tenha sido autorizada por lei federal e que preencham os requisitos dos incisos I e II do art. 213 da Constituição Federal, de 1988, que são:
a) comprovação de finalidade não-lucrativa e aplicação dos excedentes financeiros em educação;
b) assegurar a destinação do seu patrimônio a outra escola comunitária, filantrópica ou confessional, ou ao Poder Público, no caso de encerramento de suas atividades.
A sua dedutibilidade está limitada a 1,5% (um e meio por cento) do lucro operacional, antes de computada esta dedução e a das doações a entidades civis.",expense,,l10n_br_account_chart_template
account_template_30101070111,3.01.01.07.01.11,"(-) Doações a Entidades Civis","Contas que registram as doações efetuadas a:
a) entidades civis, legalmente constituídas no Brasil, sem fins lucrativos, que prestem serviços gratuitos em benefício de empregados da pessoa jurídica doadora, e respectivos dependentes, ou em benefício da comunidade na qual atuem; e
b) Organizações da Sociedade Civil de Interesse Público (OSCIP), qualificadas segundo as normas estabelecidas na Lei nº 9.790, de 23 de março de 1999.
Para fins de dedução na apuração do lucro real, as referidas doações estão limitadas a 2% (dois por cento) do lucro operacional da pessoa jurídica, antes de computadas essas deduções, observadas as seguintes regras:
a) as doações, quando em dinheiro, devem ser feitas mediante crédito em conta corrente bancária diretamente em nome da entidade beneficiária;
b) a pessoa jurídica doadora deve manter em arquivo, à disposição da fiscalização, declaração, segundo modelo aprovado pela IN SRF nº 87, de 31 de dezembro de 1996, fornecida pela entidade beneficiária, em que está se compromete a aplicar integralmente os recursos recebidos na realização de seus objetivos sociais, com identificação da pessoa física responsável pelo seu cumprimento, e a não distribuir lucros, bonificações ou vantagens a dirigentes, mantenedores ou associados, sob nenhuma forma ou pretexto (Lei nº 9.249, de 1995, art. 13, § 2º, inciso III, alínea b);
Atenção:
1) A condição estabelecida no item b não alcança a hipótese de remuneração de dirigente em decorrência de vínculo empregatício, pelas Organizações da Sociedade Civil de Interesse Público (OSCIP), qualificadas segundo as normas estabelecidas na Lei nº 9.790, de 1999, e pelas Organizações Sociais (OS), qualificadas consoante os dispositivos da Lei nº 9.637, de 15 de maio de 1998.
2) O disposto no item anterior aplica-se somente à remuneração não superior, em seu valor bruto, ao limite estabelecido para a remuneração de servidores do Poder Executivo Federal.
c) a entidade civil beneficiária deve ser reconhecida de utilidade pública por ato formal de órgão competente da União.
Atenção: O disposto neste item não se aplica às OSCIP.
d) a dedutibilidade fica condicionada a que a entidade beneficiária tenha sua condição de utilidade pública ou de OSCIP renovada anualmente pelo órgão competente da União, mediante ato formal.
Atenção: Essa renovação:
a) somente será concedida a entidade que comprove, perante o órgão competente da União, ter cumprido, no ano-calendário anterior ao do pedido, todas as exigências e condições estabelecidas;
b) produzirá efeitos para o ano-calendário subsequente ao de sua formalização.
O valor que exceder o limite permitido deve ser adicionado ao Lucro Real. ",expense,,l10n_br_account_chart_template
account_template_30101070112,3.01.01.07.01.12,"(-) Outras Contribuições, Doações e Patrocínios","Contas que registram as doações feitas, entre outras, aos Fundos controlados pelos Conselhos Municipais, Estaduais e Nacional dos Direitos da Criança e do Adolescente e Atividades de Caráter Desportivo. O valor dessas doações aos Fundos dos Direitos da Criança e do Adolescente e Atividade de Caráter Desportivo não é dedutível como despesa operacional na determinação do lucro real e da base de cálculo da contribuição social sobre o lucro líquido, mas pode ser deduzido diretamente do imposto devido.
O valor indicado nesta linha deve, também, ser adicionado ao Lucro Real.
Atenção:
1) Os valores das doações e patrocínios de caráter cultural e artístico, das doações a instituições de ensino e pesquisa e das doações a entidades civis (Lei nº 9.249, de 1995, art. 13, § 2º), devem ser indicados nas respectivas contas.
2) O valor da contribuição sindical deve ser informado na conta de ""Outras Despesas Operacionais"".",expense,,l10n_br_account_chart_template
account_template_30101070113,3.01.01.07.01.13,"(-) Alimentação do Trabalhador","Contas que registram o valor das despesas com alimentação do pessoal não ligado à produção, realizadas durante o período de apuração, ainda que a pessoa jurídica não tenha Programa de Alimentação do Trabalhador aprovado pelo Ministério do Trabalho. As despesas correspondentes, inclusive com cestas básicas de alimentos, somente podem ser dedutíveis quando a pessoa jurídica fornecer alimentação, indistintamente, a todos os seus empregados. ",expense,,l10n_br_account_chart_template
account_template_30101070114,3.01.01.07.01.14,"(-) PIS/PASEP","Contas que registram a parcela das Contribuições para o PIS/PASEP incidente sobre as demais receitas operacionais.",expense,,l10n_br_account_chart_template
account_template_30101070115,3.01.01.07.01.15,"(-) COFINS","Contas que registram a parcela da COFINS incidente sobre as demais receitas operacionais",expense,,l10n_br_account_chart_template
account_template_30101070116,3.01.01.07.01.16,"(-) Demais Impostos, Taxas e Contribuições, exceto IR e CSLL","Contas que registram os demais tributos e contribuições. Os valores indicados nesta conta são dedutíveis, para efeito de determinação do lucro real, no período de apuração em que ocorrer o fato gerador.
Não devem ser incluídas as importâncias:
a) incorporadas ao custo de bens do ativo não-circulante, exceto realizável a longo prazo;
b) correspondentes aos impostos não recuperáveis, incorporados ao custo das matérias-primas, materiais secundários, materiais de embalagem e mercadorias destinadas à revenda;
c) correspondentes aos impostos recuperáveis;
d) correspondentes aos impostos e contribuições redutores da receita bruta;
e) correspondentes às Contribuições para o PIS/PASEP e à COFINS incidentes sobre as demais receitas operacionais;
f) correspondentes à contribuição social sobre o lucro líquido e ao imposto de renda devidos. ",expense,,l10n_br_account_chart_template
account_template_30101070117,3.01.01.07.01.17,"(-) Arrendamento Mercantil","Contas que registram as despesas, não computadas nos custos, pagas ou creditadas a título de contraprestação de arrendamento mercantil, decorrentes de contrato celebrado com observância da Lei nº 6.099, de 12 de setembro de 1974, com as alterações da Lei nº 7.132, de 26 de outubro de 1983, e da Portaria MF nº 140, de 1984.
Atenção: As despesas relativas ao arrendamento de bens que não sejam intrinsecamente vinculados com a comercialização de bens ou serviços devem ser adicionadas ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_30101070118,3.01.01.07.01.18,"(-) Aluguéis","Contas que registram as despesas com aluguéis não decorrentes de arrendamento mercantil.
Atenção: As despesas relativas a aluguéis de bens móveis ou imóveis que não sejam intrinsecamente relacionados com a comercialização dos bens ou serviços devem ser adicionadas ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_30101070119,3.01.01.07.01.19,"(-) Despesas com Veículos e de Conservação de Bens e Instalações","Contas que registram as despesas relativas aos bens que não estejam ligados diretamente à produção, as realizadas com reparos que não impliquem aumento superior a um ano da vida útil do bem, prevista no ato de sua aquisição, e as relativas a combustíveis e lubrificantes para veículos.
Atenção: Somente são dedutíveis, para fins de apuração do lucro real, as despesas com Veículos e de Conservação de Bens e Instalações relativas a bens intrinsecamente vinculados com a comercialização de bens ou serviços. Os gastos considerados indedutíveis devem ser adicionados ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_30101070120,3.01.01.07.01.20,"(-) Propaganda, Publicidade e Patrocínio","Contas que registram as despesas com propaganda, publicidade e patrocínio.
Atenção: Essas despesas são dedutíveis nas condições estabelecidas no art. 366 do Decreto nº 3.000, de 1999, segundo o regime de competência.",expense,,l10n_br_account_chart_template
account_template_30101070121,3.01.01.07.01.21,"(-) Propaganda, Publicidade e Patrocínio de Assoc. Desportivas que Mantenha Equipe de Futebol Profissional","Contas que registram as despesas com propaganda e publicidade e patrocínios destinados a manutenção de Equipes de Futebol profissional.",expense,,l10n_br_account_chart_template
account_template_30101070122,3.01.01.07.01.22,"(-) Multas","Contas que registram as despesas com multas.                                                                                                                                                                                                                                                                         São totalmente indedutíveis não só as multas impostas por infrações fiscais de que resulte falta ou insuficiência de pagamento de tributo ou contribuição, como também aquelas que decorram de infrações a normas não tributárias (multas de trânsito, por exemplo). São dedutíveis as multas fiscais de natureza compensatória e aquelas impostas por descumprimento de obrigações tributárias, meramente acessórias, de que não resulte falta ou insuficiência de pagamento de tributo ou contribuição (PN CST nº 61, de 1979).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 Atenção: Os valores das multas indedutíveis devem ser adicionados ao Lucro Real.                                                      ",expense,,l10n_br_account_chart_template
account_template_30101070123,3.01.01.07.01.23,"(-) Encargos de Depreciação","Contas que registram o valor de depreciação com bens, inclusive bens adquiridos sob a modalidade de arrendamento financeiro, não aplicados diretamente na produção.
Atenção: Somente são dedutíveis, para fins de apuração do lucro real, as despesas com Encargos de Depreciação de Bens e Instalações intrinsecamente vinculados com a comercialização de bens ou serviços. Os gastos considerados indedutíveis devem ser adicionados ao Lucro Real.",expense_depreciation,,l10n_br_account_chart_template
account_template_30101070124,3.01.01.07.01.24,"(-) Encargos de Amortização","Contas que registram o valor de amortização de direitos ou bens intangíveis, inclusive objeto de arrendamento financeiro, não aplicados diretamente na produção.
Atenção: Somente são dedutíveis, para fins de apuração do lucro real, as despesas com Encargos de Amortização de Bens e Instalações intrinsecamente vinculados com a comercialização de bens ou serviços. Os gastos considerados indedutíveis devem ser adicionados ao Lucro Real.",expense_depreciation,,l10n_br_account_chart_template
account_template_30101070125,3.01.01.07.01.25,"(-) Perdas em Operações de Crédito","Contas que registram as perdas efetivas no recebimento de créditos decorrentes das atividades da pessoa jurídica.",expense,,l10n_br_account_chart_template
account_template_30101070126,3.01.01.07.01.26,"(-) Provisões para Férias","Contas que registram as despesas com a constituição de provisão para o pagamento de remuneração correspondente a férias e adicional de férias de empregados, inclusive encargos sociais (Decreto nº 3.000, de 1999, art. 337, e PN CST nº 7, de 1980).",expense,,l10n_br_account_chart_template
account_template_30101070127,3.01.01.07.01.27,"(-) Provisões para 13º Salário de Empregados","Contas que registram as despesas com a constituição de provisão para 13º salário, no caso de apuração trimestral do imposto, inclusive encargos sociais (Decreto nº 3.000, de 1999, art. 338).",expense,,l10n_br_account_chart_template
account_template_30101070128,3.01.01.07.01.28,"(-) Provisão para Perda de Estoque de Livros","Contas que registram as despesas com a constituição de provisão para perda de estoque de livros.  As pessoas jurídicas que exerçam as atividades de editor (a pessoa física ou jurídica que adquire o direito de reprodução de livros, dando a eles tratamento adequado à leitura), distribuidor (a pessoa jurídica que opera no ramo de compra e venda de livros por atacado) e livreiro (a pessoa jurídica ou representante comercial autônomo que se dedica à venda de livros), poderão indicar nesta linha, a provisão para perda de estoques, calculada no último dia de cada período de apuração do imposto de renda e da contribuição social sobre o lucro líquido, correspondente a 1/3 (um terço) do valor do estoque existente naquela data, na forma da IN SRF nº 412, de 23 de março de 2004. Ao fim de cada exercício financeiro legal será feito o ajustamento da provisão dos respectivos estoques.",expense,,l10n_br_account_chart_template
account_template_30101070129,3.01.01.07.01.29,"(-) Demais Provisões","Contas que registram às despesas com provisões não relacionadas nas linhas anteriores, constituídas no decorrer do período de apuração.
Atenção: Os valores indicados nesta linha são totalmente indedutíveis, devendo ser adicionados ao Lucro Real",expense,,l10n_br_account_chart_template
account_template_30101070130,3.01.01.07.01.30,"(-) Gratificações a Administradores","Contas que registram as gratificações a administradores.
Os pagamentos e créditos a esse título são totalmente indedutíveis. Por isso, seu montante deve ser adicionado ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_30101070131,3.01.01.07.01.31,"(-) Royalties e Assistência Técnica - no PAÍS","Contas que registram as despesas correspondentes às importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no Brasil, a título de royalties e assistência técnica, científica ou assemelhada, que não estejam relacionados com a produção de bens e/ou serviços.",expense,,l10n_br_account_chart_template
account_template_30101070132,3.01.01.07.01.32,"(-) Royalties e Assistência Técnica - no EXTERIOR","Contas que registram as despesas correspondentes às importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no exterior, a título de royalties e assistência técnica, científica ou assemelhada, que não estejam relacionados com a produção de bens e/ou serviços.",expense,,l10n_br_account_chart_template
account_template_30101070133,3.01.01.07.01.33,"(-) Assistência Médica, Odontológica e Farmacêutica a Empregados","Contas que registram as despesas com assistência médica, odontológica e farmacêutica.
Atenção: O valor referente à contratação de serviços de profissionais liberais sem vínculo empregatício ou de sociedades civis deve ser informado nas contas Prestação de Serviços por Pessoa Física sem Vínculo Empregatício ou Prestação de Serviço Pessoa Jurídica, conforme o caso.",expense,,l10n_br_account_chart_template
account_template_30101070134,3.01.01.07.01.34,"(-) Pesquisas Científicas e Tecnológicas","Contas que registram as despesas efetuadas a esse título, inclusive a contrapartida das amortizações daquelas registradas no ativo diferido",expense,,l10n_br_account_chart_template
account_template_30101070135,3.01.01.07.01.35,"(-) Bens de Pequeno Valor Unitário ou de Vida Útil de até um Ano Deduzidos como Despesa","Contas que registram o valor de aquisição de bens do ativo imobilizado cujo prazo de vida útil não ultrapasse um ano, ou, caso exceda esse prazo, tenha valor unitário igual ou inferior a R$ 1 200,00 (mil duzentos reais) (Lei nº 12.973, de 2014, art. 15).",expense,,l10n_br_account_chart_template
account_template_30101070136,3.01.01.07.01.36,"(-) Despesas com Energia Elétrica","Contas que registram as despesas com energia elétrica.",expense,,l10n_br_account_chart_template
account_template_30101070137,3.01.01.07.01.37,"(-) Despesas com Água e Esgoto","Contas que registram as despesas com água e esgoto.",expense,,l10n_br_account_chart_template
account_template_30101070138,3.01.01.07.01.38,"(-) Despesas com Telefone e Internet","Contas que registram as despesas com telefone e internet.",expense,,l10n_br_account_chart_template
account_template_30101070139,3.01.01.07.01.39,"(-) Despesas com Correios e Malotes","Contas que registram as despesas com correios e malotes.",expense,,l10n_br_account_chart_template
account_template_30101070140,3.01.01.07.01.40,"(-) Despesas com Seguros","Contas que registram as despesas com seguros.",expense,,l10n_br_account_chart_template
account_template_30101070141,3.01.01.07.01.41,"(-) Benefícios Previdenciários a Empregados","Contas que registram as despesas com benefícios previdenciários a empregados.",expense,,l10n_br_account_chart_template
account_template_30101070142,3.01.01.07.01.42,"(-) Fundo de Aposentadora Individual - FAPI","Contas que registram as despesas com Fundo de Aposentadoria Individual - FAPI.",expense,,l10n_br_account_chart_template
account_template_30101070143,3.01.01.07.01.43,"(-) Planos de Poupança e Investimento - PAIT","Contas que registram as despesas com Plano de Poupança e Investimento - PAIT.",expense,,l10n_br_account_chart_template
account_template_30101070144,3.01.01.07.01.44,"(-) Pesquisa e Desenvolvimento Abrangidas no Programa Rota 2030","Contas que registram as despesas com:
I - pesquisa, abrangidas as atividades de pesquisa básica dirigida, de pesquisa aplicada, de desenvolvimento experimental e de proje-tos estruturantes; e
II - desenvolvimento, abrangidas as atividades de desenvolvimento, de capacitação de fornecedores, de manufatura básica, de tecno-logia industrial básica e de serviços de apoio técnico.
(Lei nº 13.755/2018, art. 11).",expense,,l10n_br_account_chart_template
account_template_30101090101,3.01.01.09.01.01,"(-) Variações Cambiais Passivas","Contas que registram as perdas monetárias passivas resultantes da atualização dos direitos de créditos e das obrigações, calculadas com base nas variações nas taxas de câmbio (Lei nº 9.069, de 1995, art. 52, e Lei nº 9.249, de 1995, art. 8º).
Incluir, nesta linha, a variação cambial passiva correspondente:
a) à atualização das obrigações e dos créditos em moeda estrangeira, registrada em qualquer data e apurada no encerramento do período de apuração em função da taxa de câmbio vigente;
b) às operações com moeda estrangeira e conversão de obrigações para moeda nacional, ou novação dessas obrigações, ou sua extinção, total ou parcial, em virtude de capitalização, dação em pagamento, compensação, ou qualquer outro modo, desde que observadas as condições fixadas pelo Banco Central do Brasil.
Atenção: As variações cambiais passivas decorrentes dos direitos de crédito e de obrigações, em função da taxa de câmbio, são consideradas como despesa financeira, inclusive para fins de cálculo do lucro da exploração (Lei nº 9.718, art. 9º c/c art. 17).",expense,,l10n_br_account_chart_template
account_template_30101090102,3.01.01.09.01.02,"(-) Perdas Incorridas no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório das perdas incorridas, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País;
b) as perdas incorridas nas alienações, fora de bolsa, de ouro, ativo financeiro, e de participações societárias, exceto as alienações de participações societárias permanentes em sociedades coligadas e controladas e de participações societárias que permanecerem no ativo da pessoa jurídica até o término do ano-calendário seguinte ao de suas aquisições; e
c) as perdas em operações de swap e no resgate de quota de fundo de investimento que mantenha, no mínimo, 67% (sessenta e sete por cento) de ações negociadas no mercado à vista de bolsa de valores ou entidade assemelhada (Lei nº 9.532, de 1997, art. 28, alterado pela MP nº 1.636, de 1998, art. 2º, e reedições).
São consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros as entidades cujo objeto social seja análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM). Atenção:
1) As perdas apuradas nas operações de renda variável, exceto day-trade, somente são dedutíveis na determinação do lucro real até o limite dos ganhos auferidos em operações de mesma natureza, observado o disposto nos itens 3 a 5. As perdas não deduzidas em um período de apuração podem sê-lo nos períodos de apuração subsequentes. A parcela não dedutível no período de apuração deve ser controlada na Parte B do Lalur.
2) A partir de 1º de janeiro de 2000, as perdas apuradas em operações, exceto day-trade, no mercado à vista de ações somente são compensadas com os ganhos líquidos auferidos em operações, exceto day-trade, realizadas exclusivamente nesse mercado.
3) O saldo de perdas decorrentes de operações, exceto day-trade, existente em 31 de dezembro de 1999 pode ser compensado com os ganhos líquidos auferidos:
a) no mercado à vista de ações, se as perdas decorreram de operações, exceto day-trade, realizadas exclusivamente nesse mercado; e
b) em quaisquer mercados, se as perdas decorreram de operações, exceto day-trade, realizadas em mercados diversificados.
4) As limitações de realização de perdas, de que tratam as instruções de preenchimento desta linha, não se aplicam às pessoas jurídicas citadas no inciso I do art. 35 da IN SRF nº 25, de 6 de março de 2001, e às operações de swap utilizadas como cobertura (hedge).",expense,,l10n_br_account_chart_template
account_template_30101090103,3.01.01.09.01.03,"(-) Perdas em Operações Day-Trade","Contas que registram o somatório das perdas diárias apuradas, em cada mês do período de apuração, em operações day-trade.
Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado à vista, no mesmo dia.
Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia",expense,,l10n_br_account_chart_template
account_template_30101090104,3.01.01.09.01.04,"(-) Despesas de Juros sobre o Capital Próprio","Contas que registram os juros pagos ou creditados individualizadamente a titular, sócios ou acionistas, a título de remuneração do capital próprio, calculados sobre as contas do patrimônio líquido e limitados à variação, pro rata dia, da Taxa de Juros de Longo Prazo (TJLP), observando-se o regime de competência (Lei nº 9.249, de 1995, art. 9º).                                                                                                                                                                                                                                                                                                                                           Atenção: Quanto à dedutibilidade dos juros como despesa operacional, para fins de determinação do lucro real e da base de cálculo da CSLL.",expense,,l10n_br_account_chart_template
account_template_30101090105,3.01.01.09.01.05,"(-) Despesas de Remuneração de Debêntures","Contas que registram as despesas de Remuneração de Debêntures.",expense,,l10n_br_account_chart_template
account_template_30101090106,3.01.01.09.01.06,"(-) Juros com Empréstimos de Pessoas Vinculadas ou Situadas em País com Tributação favorecida","Contas que registram os juros pagos ou creditados por fonte situada no Brasil à pessoa física ou jurídica, vinculada nos termos do art. 23 da Lei nº 9.430, de 27 de dezembro de 1996, residente ou domiciliada no exterior, não constituída em país ou dependência com tributação favorecida ou sob regime fiscal privilegiado, observado o art. 24 da Lei nº 12.249, de 11 de junho de 2010.
Indicar também, os juros pagos ou creditados por fonte situada no Brasil à pessoa física ou jurídica residente, domiciliada ou constituída no exterior, em país ou dependência com tributação favorecida ou sob regime fiscal privilegiado, nos termos dos arts. 24 e 24-A da Lei nº 9.430, de 27 de dezembro de 1996, observado o art. 25 da Lei nº 12.249, de 2010.",expense,,l10n_br_account_chart_template
account_template_30101090107,3.01.01.09.01.07,"(-) Despesas Financeiras Relativas a Arrendamento","Contas que registram a contrapartida da realização do ajuste ao valor presente dos elementos monetários do passivo decorrentes de operações de longo prazo ou quando houver efeito relevante relativos a arrendamento.",expense,,l10n_br_account_chart_template
account_template_30101090108,3.01.01.09.01.08,"(-) Outras Despesas Financeiras","Contas que registram as despesas relativas a juros, não incluídas nas contas específicas, tais despesas serão obrigatoriamente apropriadas, segundo o regime de competência.
Atenção:
1) As variações monetárias passivas decorrentes da atualização das obrigações, em função de índices ou coeficientes aplicáveis por disposição legal ou contratual, devem ser informadas como despesa financeira.
2) As variações cambiais passivas não devem ser informadas nesta linha, e sim em conta específica.",expense,,l10n_br_account_chart_template
account_template_30101090109,3.01.01.09.01.09,"(-) Resultados Negativos em Participações Societárias Avaliadas pelo Método de Equivalência Patrimonial","Contas que registram as perdas por ajustes no valor de investimentos relevantes avaliados pelo método da equivalência patrimonial, decorrentes de prejuízos apurados nas controladas e coligadas. O valor indicado deve ser adicionado ao lucro líquido, para determinação do lucro real.
Atenção:
1) Considera-se controlada a filial, a agência, a sucursal, a dependência ou o escritório de representação no exterior, sempre que os respectivos ativos e passivos não estejam incluídos na contabilidade da investidora, por força de normatização específica.
2) Devem, também, ser indicados nesta linha os resultados negativos derivados de participações societárias no exterior, avaliadas pelo patrimônio líquido. Incluem-se, nestas informações, as perdas apuradas em filiais, sucursais e agências da pessoa jurídica localizadas no exterior.",expense,,l10n_br_account_chart_template
account_template_30101090110,3.01.01.09.01.10,"(-) Resultados Negativos em SCP Avaliadas pelo Método de Equivalência Patrimonial","Conta utilizada pelos sócios ostensivos, pessoas jurídicas, de sociedades em conta de participação, para indicar as perdas por ajustes no valor de participação em SCP, avaliada pelo método da equivalência patrimonial. O valor dessas perdas deve ser adicionado ao lucro líquido na determinação do lucro real ",expense,,l10n_br_account_chart_template
account_template_30101090111,3.01.01.09.01.11,"(-) Perdas em Operações Realizadas no Exterior","Contas que registram as perdas em operações realizadas no exterior diretamente pela pessoa jurídica domiciliada no Brasil, com exceção das perdas de capital decorrentes da alienação de bens e direitos do ativo permanente situados no exterior, que devem ser indicadas em conta específica. O valor aqui indicado deve ser adicionado ao lucro líquido para fins de apuração do lucro real",expense,,l10n_br_account_chart_template
account_template_30101090112,3.01.01.09.01.12,"(-) Perdas Estimadas Decorrentes de Teste de Recuperabilidade (Impairment)","Contas que registram a redução dos valores registrados no ativo decorrentes de análise sobre a recuperação dos ativos (teste de recuperabilidade).",expense,,l10n_br_account_chart_template
account_template_30101090113,3.01.01.09.01.13,"(-) Despesas de Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram as contrapartidas da realização do grupo Ajustes de Avaliação Patrimonial do PL reconhecidas no resultado do exercício em obediência ao regime de competência.",expense,,l10n_br_account_chart_template
account_template_30101090114,3.01.01.09.01.14,"(-) Despesas de Reclassificação de Ajustes de Avaliação Patrimonial -Reflexo","Contas que registram as contrapartidas da realização do grupo Ajustes de Avaliação Patrimonial do PL- reflexos reconhecidas no resultado do exercício em obediência ao regime de competência.",expense,,l10n_br_account_chart_template
account_template_30101090115,3.01.01.09.01.15,"(-) Despesas Financeiras Decorrentes dos Ajustes ao Valor Presente","Contas que registram a realização do ajuste ao valor presente dos elementos monetários do passivo decorrentes de operações de longo prazo ou quando houver efeito relevante.",expense,,l10n_br_account_chart_template
account_template_30101090116,3.01.01.09.01.16,"(-) Encargos de Depreciação de Bens Objeto de Arrendamento","Contas que registram os encargos de depreciação de bens objeto de arrendamento.",expense_depreciation,,l10n_br_account_chart_template
account_template_30101090117,3.01.01.09.01.17,"(-) Encargos de Amortização de Mais -Valia","Contas que registram os encargos de amortização de mais-valia.",expense_depreciation,,l10n_br_account_chart_template
account_template_30101090118,3.01.01.09.01.18,"(-) Aluguéis de Bens Imóveis- Locador  Parte Relacionada","Contas que registram os aluguéis de bens imóveis a parte relacionada",expense,,l10n_br_account_chart_template
account_template_30101090119,3.01.01.09.01.19,"(-) Aluguéis de Bens Imóveis Locador  Parte Não Relacionada","Contas que registram os aluguéis de bens imóveis a parte não relacionada",expense,,l10n_br_account_chart_template
account_template_30101090120,3.01.01.09.01.20,"(-) Despesas com Empréstimos de Valores Mobiliários","Contas que registram as despesas com empréstimos de valores mobiliários.",expense,,l10n_br_account_chart_template
account_template_30101090121,3.01.01.09.01.21,"(-) Despesas com Corretagem e Emolumentos","Contas que registram as despesas com corretagem e emolumentos.",expense,,l10n_br_account_chart_template
account_template_30101090122,3.01.01.09.01.22,"(-) Despesas  com Deságio na Cessão de Títulos","Contas que registram as despesas com deságio na cessão de títulos decorrentes de securitização",expense,,l10n_br_account_chart_template
account_template_30101090123,3.01.01.09.01.23,"(-) Despesas Incorridas em Operações de Mútuo – Parte Relacionada","Contas que registram os juros incorridos em operações de mútuo com partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",expense,,l10n_br_account_chart_template
account_template_30101090124,3.01.01.09.01.24,"(-) Despesas Incorridas em Operações de Mútuo – Parte Não Relacionada","Contas que registram os juros incorridos em operações de mútuo com partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",expense,,l10n_br_account_chart_template
account_template_30101090125,3.01.01.09.01.25,"(-) Despesas Incorridas  em Outros Passivos Financeiros Mensurados Pelo Custo Amortizado","Contas que registram os juros auferidos com passivos financeiros mensurados pelo custo amortizado.",expense,,l10n_br_account_chart_template
account_template_30101090126,3.01.01.09.01.26,"(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros para Negociação - Não Hedge - Valor Justo pelo Resultado","Contas que registram as perdas decorrentes de ajustes a valor justo de instrumentos financeiros para negociação, exceto hedge, avaliados a valor justo pelo resultado.",expense,,l10n_br_account_chart_template
account_template_30101090127,3.01.01.09.01.27,"(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros Disponíveis para Venda - Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram as perdas decorrentes de ajustes a valor justo de instrumentos financeiros disponíveis para venda no momento da reclassificação dos ajustes de avaliação patrimonial.",expense,,l10n_br_account_chart_template
account_template_30101090128,3.01.01.09.01.28,"(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros de Hedge de Valor Justo","Contas que registram as perdas decorrentes de ajustes a valor justo de instrumentos financeiros de hedge.",expense,,l10n_br_account_chart_template
account_template_30101090129,3.01.01.09.01.29,"(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros de Hedge -  Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram as perdas decorrentes de ajustes a valor justo de instrumentos financeiros de hedge no momento da reclassificação dos ajustes de avaliação patrimonial.",expense,,l10n_br_account_chart_template
account_template_30101090130,3.01.01.09.01.30,"(-) Perda de Ajuste a Valor Justo - Item Objeto de Hedge de Valor Justo","Contas que registram as perdas decorrentes de ajustes a valor justo de item objeto de hedge.",expense,,l10n_br_account_chart_template
account_template_30101090131,3.01.01.09.01.31,"(-) Perda de Ajuste a Valor Justo - Propriedade para Investimento","Contas que registram as perdas decorrentes de ajustes a valor justo de propriedades para investimento.",expense,,l10n_br_account_chart_template
account_template_30101090132,3.01.01.09.01.32,"(-) Perda de Ajuste a Valor Justo - Ativo Biológico Consumível","Contas que registram as perdas decorrentes de ajustes a valor justo de ativo biológico consumível.",expense,,l10n_br_account_chart_template
account_template_30101090133,3.01.01.09.01.33,"(-) Perda de Ajuste a Valor Justo - Ativo Biológico de Produção","Contas que registram as perdas decorrentes de ajustes a valor justo de ativo biológico de produção.",expense,,l10n_br_account_chart_template
account_template_30101090134,3.01.01.09.01.34,"(-) Perda de Ajuste a Valor Justo - Ativos Não Circulantes Mantidos para Venda","Contas que registram as perdas decorrentes de ajustes a valor justo de ativos não circulantes mantidos para venda.",expense,,l10n_br_account_chart_template
account_template_30101090135,3.01.01.09.01.35,"(-) Perda de Ajuste a Valor Justo - Subscrição de Capital com demais Bens","Contas que registram as perdas decorrentes de ajustes a valor justo de subscrição de capital com demais bens.",expense,,l10n_br_account_chart_template
account_template_30101090136,3.01.01.09.01.36,"(-) Perda de Ajuste a Valor Justo - Subscrição de Capital com Participação Societária","Contas que registram as perdas decorrentes de ajustes a valor justo de subscrição de capital com participação societária.",expense,,l10n_br_account_chart_template
account_template_30101090137,3.01.01.09.01.37,"(-) Perda de Ajuste a Valor Justo - Aquisição de Participação Societária em Estágios","Contas que registram as perdas decorrentes de ajustes a valor justo de aquisição de participação societária em estágios.",expense,,l10n_br_account_chart_template
account_template_30101090138,3.01.01.09.01.38,"(-) Perda de Ajuste a Valor Justo - Decorrente de Permuta de Ativos ou Passivos","Contas que registram as perdas decorrentes de ajustes a valor justo devido a permuta de ativos ou passivos.",expense,,l10n_br_account_chart_template
account_template_30101090139,3.01.01.09.01.39,"(-) Perda de Ajuste a Valor Justo - Outras Operações","Contas que registram as perdas decorrentes de ajustes a valor justo de outras operações não classificáveis neste plano de contas.",expense,,l10n_br_account_chart_template
account_template_30101090199,3.01.01.09.01.99,"(-) Outras Despesas Operacionais","Contas que registram as demais despesas que, por definição legal, sejam consideradas operacionais, não enquadráveis contas específicas.",expense,,l10n_br_account_chart_template
account_template_30101110101,3.01.01.11.01.01,"Receitas na Alienação de Participações Integrantes do Ativo Circulante ou do Ativo Realizável a Longo Prazo","Contas que registram as receitas auferidas quando da conclusão da alienação dos ativos destinados a vendas.  Esses ativos ou grupo de ativos foram reclassificados para o circulante quando da decisão da administração da companhia na venda dos ativos ou grupo de ativos.",income_other,,l10n_br_account_chart_template
account_template_30101110102,3.01.01.11.01.02,"Receitas de Alienações de Bens e Direitos do Ativo Não Circulante Investimentos, Imobilizado e Intangível","Contas que registram as receitas auferidas por meio de alienações, inclusive por desapropriação de bens e direitos classificados em investimentos, imobilizado e intangível.",income_other,,l10n_br_account_chart_template
account_template_30101110103,3.01.01.11.01.03,"Ganhos de Capital por Variação Percentual em Participação Societária Avaliada pelo Patrimônio Líquido","Contas que registram o ganho de capital resultante de acréscimo, por variação percentual, do valor do patrimônio líquido de investimento avaliado pelo método da equivalência patrimonial.",income_other,,l10n_br_account_chart_template
account_template_30101110104,3.01.01.11.01.04,"(-) Valor Contábil de Participações Integrantes do Ativo Circulante ou do Ativo Realizável a Longo Prazo Alienadas","Contas que registram o valor contábil quando da conclusão da alienação dos ativos destinados a vendas.  Esses ativos ou grupo de ativos foram reclassificados para o circulante quando da decisão da administração da companhia na venda dos ativos ou grupo de ativos.",expense,,l10n_br_account_chart_template
account_template_30101110105,3.01.01.11.01.05,"(-) Valor Contábil dos Bens e Direitos do Ativo Não Circulante Investimentos, Intangível e Imobilizado Alienados","Contas que registram o valor contábil por meio de alienações, inclusive por desapropriação de bens e direitos classificados em investimentos, imobilizado e intangível.",expense,,l10n_br_account_chart_template
account_template_30101110106,3.01.01.11.01.06,"(-) Perdas de Capital por Variação Percentual em Participação Societária Avaliada pelo Patrimônio Líquido","Contas que registram a perda de capital resultante de decréscimo, por variação percentual, do valor do patrimônio líquido de investimento avaliado pelo método da equivalência patrimonial.",expense,,l10n_br_account_chart_template
account_template_30101110107,3.01.01.11.01.07,"Receitas de Operações Descontinuadas","Contas que registram as receitas de alienação de operações descontinuadas.",income_other,,l10n_br_account_chart_template
account_template_30101110108,3.01.01.11.01.08,"(-) Despesas de Operações Descontinuadas","Contas que registram o valor contábil na alienação de operações descontinuadas.",expense,,l10n_br_account_chart_template
account_template_30105010101,3.01.05.01.01.01,"(-) Participações de Empregados","Contas que registram as participações atribuídas a empregados segundo disposição legal, estatutária, contratual ou por deliberação da assembleia de acionistas ou sócios.
Para efeito de apuração do lucro real, somente são dedutíveis as participações atribuídas indiscriminadamente a todos os empregados que se encontrem na mesma situação de emprego, e desde que atendidos os demais requisitos legais definidos na Lei nº 10.101, de 19 de dezembro de 2000.
Atenção: É vedado qualquer pagamento de antecipação ou qualquer distribuição de valores a título de participação nos lucros ou resultados da empresa em periodicidade inferior a um semestre civil, ou mais de duas vezes no mesmo ano civil.",expense,,l10n_br_account_chart_template
account_template_30105010102,3.01.05.01.01.02,"(-) Contribuições para Assistência ou Previdência de Empregados","Contas que registram as contribuições para instituições ou fundos de assistência ou previdência de empregados, baseadas nos lucros. Para efeito do imposto de renda, essas contribuições somente podem ser deduzidas quando pagas a entidades de previdência privada expressamente autorizadas a funcionar. As contribuições que não satisfaçam as condições legais devem ser adicionadas ao Lucro Real. Não indicar, nesta linha, aquelas contribuições já deduzidas como custo ou despesa operacional.",expense,,l10n_br_account_chart_template
account_template_30105010198,3.01.05.01.01.98,"(-) Outras Participações de Empregados","Contas que registram as demais participações de empregados.",expense,,l10n_br_account_chart_template
account_template_30105010301,3.01.05.01.03.01,"(-) Participações de Administradores e Partes Beneficiárias","Contas que registram as participações nos lucros atribuídas a administradores, sócio, titular de empresa individual e a portadores de partes beneficiárias, durante o período de apuração.",expense,,l10n_br_account_chart_template
account_template_30105010302,3.01.05.01.03.02,"(-) Participações de Debêntures","Contas que registram as participações nos lucros da companhia atribuídas a debêntures de sua emissão.",expense,,l10n_br_account_chart_template
account_template_30105010398,3.01.05.01.03.98,"(-) Outras Participações","Contas que registram as outras participações não especificadas anteriormente.",expense,,l10n_br_account_chart_template
account_template_30201010101,3.02.01.01.01.01,"(-) Provisão para Contribuição Social sobre o Lucro Líquido (Atividade Geral)","Contas que registram a soma das provisões para a CSLL calculadas sobre a base de cálculo correspondente ao período de apuração da atividade geral. A sua constituição é obrigatória para todas as pessoas jurídicas tributadas com base no lucro real.
As cooperativas devem informar, nesta linha, a provisão da CSLL sobre os resultados das operações realizadas com os não-associados",expense,,l10n_br_account_chart_template
account_template_30201010102,3.02.01.01.01.02,"(-) Provisão para Imposto de Renda - Pessoa Jurídica (Atividade Geral e Rural)","Contas que registram a soma das provisões para o imposto de renda constituídas sobre o lucro real. A sua constituição é obrigatória para todas as pessoas jurídicas tributadas com base no lucro real.",expense,,l10n_br_account_chart_template
account_template_30201010111,3.02.01.01.01.11,"(-) Provisão para Contribuição Social sobre o Lucro Líquido - Lucros Diferidos (Atividade Geral)","Contas que registram a soma das provisões para a CSLL calculadas sobre os lucros diferidos da atividade geral, se for o caso. A sua constituição é obrigatória para todas as pessoas jurídicas tributadas com base no lucro real.
As cooperativas devem informar, nesta linha, a provisão da CSLL sobre os resultados das operações realizadas com os não-associados",expense,,l10n_br_account_chart_template
account_template_30201010112,3.02.01.01.01.12,"(-) Provisão para Imposto de Renda - Pessoa Jurídica - Lucros Diferidos (Atividade Geral e Rural)","Contas que registram a soma das provisões para o imposto de renda constituídas sobre os lucros diferidos. A sua constituição é obrigatória para todas as pessoas jurídicas tributadas com base no lucro real.",expense,,l10n_br_account_chart_template
account_template_31101010101,3.11.01.01.01.01,"Receita da Atividade Rural - Exportação Direta","Contas que registram o valor da receita auferida em decorrência da exportação direta de mercadorias e produtos da atividade rural. ",income,,l10n_br_account_chart_template
account_template_31101010102,3.11.01.01.01.02,"Receita da Atividade Rural - Venda a Comercial Exportadora com Fim Específico de Exportação","Contas que registram o valor da receita auferida em decorrência da venda de mercadorias e produtos da atividade rural a empresa comercial exportadora, com fim específico de exportação.",income,,l10n_br_account_chart_template
account_template_31101010103,3.11.01.01.01.03,"Receita da Atividade Rural - Mercado Interno","Contas que registram a receita auferida no mercado interno correspondente à venda mercadorias e produtos da atividade rural. (Não se incluem o valor correspondente ao Imposto sobre Produtos Industrializados (IPI) cobrado destacadamente do comprador ou contratante, uma vez que o vendedor é mero depositário e este imposto não integra o preço de venda da mercadoria, e, também, o valor correspondente ao ICMS cobrado na condição de substituto).",income,,l10n_br_account_chart_template
account_template_31101010201,3.11.01.01.02.01,"(-) Vendas Canceladas e Devoluções de Vendas","Contas que registram o valor que correspondam as vendas canceladas e a devoluções de vendas.",expense,,l10n_br_account_chart_template
account_template_31101010202,3.11.01.01.02.02,"(-) Descontos Incondicionais e Abatimentos","Contas que registram o valor que corresponde a descontos incondicionais e abatimentos concedidos.",expense,,l10n_br_account_chart_template
account_template_31101010203,3.11.01.01.02.03,"(-) ICMS","Contas que registram o total do Imposto Sobre Operações Relativas à Circulação de Mercadorias e Sobre Prestação de Serviços de Transporte Interestadual e Intermunicipal e de Comunicação (ICMS) calculado sobre as receitas das vendas e de serviços.  Informar o resultado da aplicação das alíquotas sobre as respectivas receitas, e não o montante recolhido, durante o período de apuração, pela pessoa jurídica.  O valor referente ao ICMS pago como substituto não deve ser incluído nesta conta.",expense,,l10n_br_account_chart_template
account_template_31101010204,3.11.01.01.02.04,"(-) Cofins Sobre Receita Bruta","Contas que registram o valor total da COFINS apurada sobre a receita de vendas em consonância com a legislação vigente à época da ocorrência dos fatos geradores. O valor informado deve ser apurado de forma centralizada pelo estabelecimento matriz, quando a pessoa jurídica possuir mais de um estabelecimento (Lei nº 9.779, de 1999, art. 15, III).  Não incluir a COFINS incidente sobre as demais receitas operacionais, que deverá ser informada em conta distinta.",expense,,l10n_br_account_chart_template
account_template_31101010205,3.11.01.01.02.05,"(-) PIS/Pasep Sobre Receita Bruta","Contas que registram o valor total das contribuições para o PIS/PASEP apurado sobre a receita de vendas em consonância com a legislação vigente à época da ocorrência dos fatos geradores. O valor informado deve ser apurado de forma centralizada pelo estabelecimento matriz, quando a pessoa jurídica possuir mais de um estabelecimento (Lei nº 9.779, de 1999, art. 15, III). Não incluir a COFINS incidente sobre as demais receitas operacionais, que deverá ser informada em conta distinta.",expense,,l10n_br_account_chart_template
account_template_31101010206,3.11.01.01.02.06,"(-) ISS","Contas que registram o Imposto sobre Serviço de qualquer Natureza (ISS) relativo às receitas de serviços, conforme legislação específica.",expense,,l10n_br_account_chart_template
account_template_31101010209,3.11.01.01.02.09,"(-) Demais Impostos e Contribuições Incidentes sobre Vendas e Serviços","Contas que registrem os demais impostos e contribuições incidentes sobre as receitas das vendas de que tratam as contas integrantes do grupo RECEITA BRUTA, que guardem proporcionalidade com o preço e sejam considerados redutores das receitas de vendas.",expense,,l10n_br_account_chart_template
account_template_31101010210,3.11.01.01.02.10,"(-) Ajuste a Valor Presente sobre Receita Bruta","Contas que registram os expurgos dos efeitos do ajuste a valor presente sobre a receita bruta.",expense,,l10n_br_account_chart_template
account_template_31101030101,3.11.01.03.01.01,"(-) Custo dos Bens e Produtos Vendidos da Atividade Rural","Contas que registram o valor dos gastos que compõem o custo total de produção própria após a realização dos estoques.",expense,,l10n_br_account_chart_template
account_template_31101050101,3.11.01.05.01.01,"Variações Cambiais Ativas","Contas que registram os ganhos apurados em razão de variações ativas decorrentes da atualização dos direitos de crédito e obrigações, calculados com base nas variações nas taxas de câmbio.",income_other,,l10n_br_account_chart_template
account_template_31101050102,3.11.01.05.01.02,"Ganhos Auferidos no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório dos ganhos auferidos, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País;
Atenção:
1) Os ganhos auferidos em operações day-trade devem ser informados em conta específica.
2) O valor correspondente às perdas incorridas no mercado de renda variável, exceto day-trade, deve ser informado em conta específica.
3) São consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros as entidades cujo objeto social seja análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM).",income_other,,l10n_br_account_chart_template
account_template_31101050103,3.11.01.05.01.03,"Ganhos em Operações Day-Trade","Contas que registram os ganhos diários auferidos, em cada mês do período de apuração, em operações day-trade.  Considera-se ganho o resultado positivo auferido nas operações citadas acima, realizadas em cada mês, admitida a dedução dos custos e despesas incorridos, necessários à realização das operações.  Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado à vista, no mesmo dia.  Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia.",income_other,,l10n_br_account_chart_template
account_template_31101050104,3.11.01.05.01.04,"Receitas de Juros sobre o Capital Próprio","Contas que registram os juros recebidos, a título de remuneração do capital próprio, em conformidade com o art. 9º da Lei nº 9.249, de 1995. O valor informado deve corresponder ao total dos juros recebidos antes do desconto do imposto de renda na fonte.
O valor do imposto de renda retido na fonte, para as pessoas jurídicas tributadas pelo lucro real, é considerado antecipação do imposto devido no encerramento do período de apuração ou, ainda, pode ser compensado com aquele que for retido, pela beneficiária, por ocasião do pagamento ou crédito de juros a título de remuneração do capital próprio, ao seu titular ou aos seus sócios. ",income_other,,l10n_br_account_chart_template
account_template_31101050105,3.11.01.05.01.05,"Outras Receitas Financeiras","Contas que registram receitas auferidas no período de apuração relativas a juros, descontos, lucro na operação de reporte, prêmio de resgate de títulos ou debêntures e rendimento nominal auferido em aplicações financeiras de renda fixa, não incluídas em linhas específicas. As receitas dessa natureza, derivadas de operações com títulos vencíveis após o encerramento do período de apuração, serão rateadas segundo o regime de competência.",income_other,,l10n_br_account_chart_template
account_template_31101050106,3.11.01.05.01.06,"Resultados Positivos em Participações Societárias Avaliadas pelo Método de Equivalência Patrimonial","Contas que registram o ganho de investimento avaliado pelo método da equivalência patrimonial.",income_other,,l10n_br_account_chart_template
account_template_31101050107,3.11.01.05.01.07,"Resultados Positivos em SCP Avaliadas pelo Método de Equivalência Patrimonial","Esta conta é utilizada pelas pessoas jurídicas que forem sócias ostensivas de sociedades em conta de participação, para a indicação:
a) de lucros derivados de participação em SCP, avaliadas pelo custo de aquisição;
b) dos ganhos por ajustes no valor de participação em SCP, avaliadas pelo método da equivalência patrimonial.
Os lucros recebidos de investimento em SCP, avaliado pelo custo de aquisição, ou a contrapartida do ajuste do investimento ao valor do patrimônio líquido da SCP, no caso de investimento avaliado por esse método, podem ser excluídos na determinação do lucro real dos sócios, pessoas jurídicas, das referidas sociedades (Decreto nº 3.000, de 1999, art. 149).",income_other,,l10n_br_account_chart_template
account_template_31101050108,3.11.01.05.01.08,"Rendimentos e Ganhos de Capital Auferidos no Exterior","Contas que registram os rendimentos e ganhos de capital auferidos no exterior diretamente pela pessoa jurídica domiciliada no Brasil, pelos seus valores antes de descontado o tributo pago no país de origem. Esses valores podem, no caso de apuração trimestral do imposto, ser excluídos na apuração do lucro real do 1º aos 3º trimestres, devendo ser adicionados ao lucro líquido na apuração do lucro real referente ao 4º trimestre.
Atenção: Os ganhos de capital referentes a alienações de bens e direitos do ativo não-circulante, exceto os classificáveis no ativo realizável a longo prazo, situados no exterior devem ser informados em outras receitas.",income_other,,l10n_br_account_chart_template
account_template_31101050109,3.11.01.05.01.09,"Reversão das Perdas Estimadas Decorrentes de Teste de Recuperabilidade (Impairment)"," Contas que registram o ganho decorrente da reversão das perdas estimadas decorrentes da aplicação de teste de recuperabilidade sobre os ativos. ",income_other,,l10n_br_account_chart_template
account_template_31101050110,3.11.01.05.01.10,"Reversão dos Saldos das Provisões","Contas que registram a reversão dos saldos não utilizados das provisões constituídas no balanço do período de apuração imediatamente anterior e ou constituída no próprio período de apuração para fins de apuração do lucro real.",income_other,,l10n_br_account_chart_template
account_template_31101050111,3.11.01.05.01.11,"Prêmios Recebidos na Emissão de Debêntures","Contas que registram valor dos prêmios recebidos na emissão de debêntures, tais como:
1) A pessoa jurídica poderá excluir o valor decorrente de prêmios recebidos na emissão de debêntures, reconhecido no exercício, para fins de apuração do lucro real; caso mantenha em reserva de lucros específica a parcela decorrente de prêmio na emissão de debêntures, apurada até o limite do lucro líquido do exercício;
2) O prêmio na emissão de debêntures será tributado caso seja dada destinação diversa da que está prevista no item 1 acima, inclusive nas hipóteses de:
a) capitalização do valor e posterior restituição de capital aos sócios ou ao titular, mediante redução do capital social, hipótese em que a base para a incidência será o valor restituído, limitado ao valor total das exclusões decorrentes de prêmios na emissão de debêntures;
b) restituição de capital aos sócios ou ao titular, mediante redução do capital social, nos 5 (cinco) anos anteriores à data da emissão das debêntures com o prêmio, com posterior capitalização do valor do prêmio, hipótese em que a base para a incidência será o valor restituído, limitado ao valor total das exclusões decorrentes de prêmios na emissão de debêntures; ou
c) integração à base de cálculo dos dividendos obrigatórios.",income_other,,l10n_br_account_chart_template
account_template_31101050112,3.11.01.05.01.12,"Doações e Subvenções para Custeio ou Operações","Contas que registram as subvenções para custeio ou operações recebidas, inclusive mediante isenção ou redução de impostos concedidas como estímulo à implantação ou expansão de empreendimentos econômicos, e as doações recebidas do Poder Público.",income_other,,l10n_br_account_chart_template
account_template_31101050113,3.11.01.05.01.13,"Receitas de Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram as contrapartidas da realização do grupo Ajustes de Avaliação Patrimonial do PL reconhecidas no resultado do exercício em obediência ao regime de competência.",income_other,,l10n_br_account_chart_template
account_template_31101050114,3.11.01.05.01.14,"Receitas de Reclassificação de Ajustes de Avaliação Patrimonial - Reflexo","Contas que registram as contrapartidas da realização do grupo Ajustes de Avaliação Patrimonial do PL - Reflexo reconhecidas no resultado do exercício em obediência ao regime de competência.",income_other,,l10n_br_account_chart_template
account_template_31101050116,3.11.01.05.01.16,"Receitas Financeiras Decorrentes de Ajustes ao Valor Presente","Contas que registram as contrapartidas de aumentos de ativos sujeitos a Ajuste ao Valor Presente de acordo com o regime de competência.",income_other,,l10n_br_account_chart_template
account_template_31101050117,3.11.01.05.01.17,"Ganho Por Compra Vantajosa em Investimentos","Contas que registram os ganhos auferidos por compra vantajosa nas aquisições de controle de investimento, independente do critério de avaliação.",income_other,,l10n_br_account_chart_template
account_template_31101050118,3.11.01.05.01.18,"Amortização de Menos-Valia","Contas que registram a amortização de menos-valia.",income_other,,l10n_br_account_chart_template
account_template_31101050119,3.11.01.05.01.19,"Receita de Aluguel de Bens Imóveis - Atividade Não Principal","Contas que registram aluguéis de bens por empresa que não tenha por objeto a locação de imóveis.",income_other,,l10n_br_account_chart_template
account_template_31101050120,3.11.01.05.01.20,"Receita de Aluguel de Bens Móveis - Atividade Não Principal","Contas que registram aluguéis de bens por empresa que não tenha por objeto a locação de móveis.",income_other,,l10n_br_account_chart_template
account_template_31101050121,3.11.01.05.01.21,"Créditos Presumidos de IPI","Contas que registram os créditos presumidos do IPI para ressarcimento do valor da Contribuição ao PIS/PASEP e COFINS.",income_other,,l10n_br_account_chart_template
account_template_31101050122,3.11.01.05.01.22,"Créditos Presumidos de PIS/COFINS","Contas que registram o crédito presumido da contribuição para o PIS/PASEP e da COFINS concedido na forma do art. 3º da Lei nº 10.147, de 2000.",income_other,,l10n_br_account_chart_template
account_template_31101050123,3.11.01.05.01.23,"Outros Créditos Fiscais Presumidos","Contas que registram outros créditos fiscais presumidos.",income_other,,l10n_br_account_chart_template
account_template_31101050124,3.11.01.05.01.24,"Multas e Outras Vantagens Recebidas","Contas que registram multas ou vantagens a título de indenização em virtude de rescisão contratual (Lei nº 9.430, de 1996, art. 70, § 3º, II).",income_other,,l10n_br_account_chart_template
account_template_31101050125,3.11.01.05.01.25,"Lucros e Dividendos Derivados de Participações Societárias Avaliadas pelo Custos de Aquisição","Contas que registram os resultados positivos em participações societárias avaliadas pelo custo de aquisição.",income_other,,l10n_br_account_chart_template
account_template_31101050126,3.11.01.05.01.26,"Receitas com Empréstimos de Valores Mobiliários","Contas que registram as receitas com empréstimos de valores mobiliários.",income_other,,l10n_br_account_chart_template
account_template_31101050127,3.11.01.05.01.27,"Rendimentos Auferidos em Operações de Mútuo – Partes Relacionadas","Contas que registram os juros auferidos em operações de mútuo com partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",income_other,,l10n_br_account_chart_template
account_template_31101050128,3.11.01.05.01.28,"Rendimentos Auferidos em Operações de Mútuo – Partes Não Relacionadas","Contas que registram os juros auferidos em operações de mútuo com partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",income_other,,l10n_br_account_chart_template
account_template_31101050129,3.11.01.05.01.29,"Rendimentos Auferidos com Debêntures - Emitente Partes  Relacionadas","Contas que registram os juros auferidos com debêntures emitidas por partes relacionadas.",income_other,,l10n_br_account_chart_template
account_template_31101050130,3.11.01.05.01.30,"Rendimentos Auferidos com Debêntures - Emitente Partes Não Relacionadas","Contas que registram os juros auferidos com debêntures emitidas por partes não relacionadas.",income_other,,l10n_br_account_chart_template
account_template_31101050131,3.11.01.05.01.31,"Rendimentos Auferidos com Títulos Públicos","Contas que registram os juros auferidos em Títulos Públicos",income_other,,l10n_br_account_chart_template
account_template_31101050132,3.11.01.05.01.32,"Juros Auferidos com Outros Ativos Financeiros Mensurados Pelo Custo Amortizado","Contas que registram os juros auferidos com outros ativos financeiros mensurados pelo custo amortizado.",income_other,,l10n_br_account_chart_template
account_template_31101050133,3.11.01.05.01.33,"Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  para Negociação - Não Hedge – Valor Justo pelo Resultado (VJPR).","Contas que registram os ganhos decorrentes de ajustes a valor justo de instrumentos financeiros para negociação, exceto hedge, avaliados a valor justo pelo resultado.",income_other,,l10n_br_account_chart_template
account_template_31101050134,3.11.01.05.01.34,"Ganho de Ajustes a Valor Justo  - Instrumentos Financeiros  Disponíveis para Venda - Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram os ganhos decorrentes de ajustes a valor justo de instrumentos financeiros disponíveis para venda no momento da reclassificação dos ajustes de avaliação patrimonial.",income_other,,l10n_br_account_chart_template
account_template_31101050135,3.11.01.05.01.35,"Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  de Hedge  de Valor Justo","Contas que registram os ganhos decorrentes de ajustes a valor justo de instrumentos financeiros de hedge.",income_other,,l10n_br_account_chart_template
account_template_31101050136,3.11.01.05.01.36,"Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  de Hedge -  Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram os ganhos decorrentes de ajustes a valor justo de instrumentos financeiros de hedge no momento da reclassificação dos ajustes de avaliação patrimonial.",income_other,,l10n_br_account_chart_template
account_template_31101050137,3.11.01.05.01.37,"Ganho de Ajustes a Valor Justo - Item Objeto de Hedge de Valor Justo","Contas que registram os ganhos decorrentes de ajustes a valor justo de item objeto de hedge.",income_other,,l10n_br_account_chart_template
account_template_31101050138,3.11.01.05.01.38,"Ganho de Ajustes a Valor Justo - Propriedade para Investimento","Contas que registram os ganhos decorrentes de ajustes a valor justo de propriedades para investimento.",income_other,,l10n_br_account_chart_template
account_template_31101050139,3.11.01.05.01.39,"Ganho de Ajustes a Valor Justo - Ativo Biológico Consumível","Contas que registram os ganhos decorrentes de ajustes a valor justo de ativo biológico consumível.",income_other,,l10n_br_account_chart_template
account_template_31101050140,3.11.01.05.01.40,"Ganho de Ajustes a Valor Justo - Ativo Biológico de Produção","Contas que registram os ganhos decorrentes de ajustes a valor justo de ativo biológico de produção.",income_other,,l10n_br_account_chart_template
account_template_31101050141,3.11.01.05.01.41,"Ganho de Ajustes a Valor Justo - Ativos Não Circulantes Mantidos para Venda","Contas que registram os ganhos decorrentes de ajustes a valor justo de ativos não circulantes mantidos para venda.",income_other,,l10n_br_account_chart_template
account_template_31101050142,3.11.01.05.01.42,"Ganho de Ajustes a Valor Justo - Subscrição de Capital com demais Bens","Contas que registram os ganhos decorrentes de ajustes a valor justo de subscrição de capital com demais bens.",income_other,,l10n_br_account_chart_template
account_template_31101050143,3.11.01.05.01.43,"Ganho de Ajustes a Valor Justo - Subscrição de Capital com Participação Societária","Contas que registram os ganhos decorrentes de ajustes a valor justo de subscrição de capital com participação societária.",income_other,,l10n_br_account_chart_template
account_template_31101050144,3.11.01.05.01.44,"Ganho de Ajustes a Valor Justo - Aquisição de Participação Societária em Estágios","Contas que registram os ganhos decorrentes de ajustes a valor justo de aquisição de participação societária em estágios.",income_other,,l10n_br_account_chart_template
account_template_31101050145,3.11.01.05.01.45,"Ganho de Ajustes a Valor Justo - Decorrente de Permuta de Ativos ou Passivos","Contas que registram os ganhos decorrentes de ajustes a valor justo devido a permuta de ativos ou passivos.",income_other,,l10n_br_account_chart_template
account_template_31101050146,3.11.01.05.01.46,"Ganho de Ajustes a Valor Justo - Outras Operações","Contas que registram os ganhos decorrentes de ajustes a valor justo de outras operações não classificáveis neste plano de contas.",income_other,,l10n_br_account_chart_template
account_template_31101050147,3.11.01.05.01.47,"Doações e Subvenções para Investimentos","Contas que registras as subvenções para investimento recebidas, inclusive mediante isenção ou redução de impostos concedidas como estímulo à implantação ou expansão de empreendimentos econômicos, e as doações recebidas do Poder Público.
Atenção:
1) A pessoa jurídica poderá excluir o valor decorrente de doações ou subvenções governamentais para investimentos, reconhecido no exercício, para fins de apuração do lucro real; caso mantenha em reserva de lucros a que se refere o art. 195-A da Lei nº 6.404, de 1976, a parcela decorrente de doações ou subvenções governamentais, apurada até o limite do lucro líquido do exercício.
2) As doações e subvenções serão tributadas caso seja dada destinação diversa da prevista no item 1, inclusive nas hipóteses de:
a) capitalização do valor e posterior restituição de capital aos sócios ou ao titular, mediante redução do capital social, hipótese em que a base para a incidência será o valor restituído, limitado ao valor total das exclusões decorrentes de doações ou subvenções governamentais para investimentos;
b) restituição de capital aos sócios ou ao titular, mediante redução do capital social, nos 5 (cinco) anos anteriores à data da doação ou da subvenção, com posterior capitalização do valor da doação ou da subvenção, hipótese em que a base para a incidência será o valor restituído, limitado ao valor total das exclusões decorrentes de doações ou de subvenções governamentais para investimentos; ou
c) integração à base de cálculo dos dividendos obrigatórios.
3) Se, no período base em que ocorrer a exclusão, a pessoa jurídica apurar prejuízo contábil ou lucro líquido contábil inferior à parcela decorrente de doações e subvenções governamentais, e neste caso não puder ser constituída como parcela de lucros nos termos do item 1 acima, esta deverá ocorrer nos exercícios subsequentes.",income_other,,l10n_br_account_chart_template
account_template_31101050199,3.11.01.05.01.99,"Outras Receitas Operacionais","Contas que registrem as demais receitas que, por definição legal, sejam consideradas operacionais. tais como recuperações de despesas operacionais de períodos de apuração anteriores, tais como: prêmios de seguros, importâncias levantadas das contas vinculadas do FGTS, ressarcimento de desfalques, roubos e furtos, etc. As recuperações de custos e despesas no decurso do próprio período de apuração devem ser creditadas diretamente às contas de resultado em que foram debitadas.",income_other,,l10n_br_account_chart_template
account_template_31101070101,3.11.01.07.01.01,"(-) Remuneração a Dirigentes e a Conselho de Administração","Contas que registram a despesa incorrida relativa à remuneração mensal e fixa atribuída ao titular de firma individual, aos sócios, diretores e administradores de sociedades, ou aos representantes legais de sociedades estrangeiras, as despesas incorridas com os salários indiretos concedidos pela empresa a administradores, diretores, gerentes e seus assessores (PN Cosit nº 11, de 1992), e o valor referente às remunerações atribuídas aos membros do conselho fiscal ou consultivo.
Atenção:
1) Os valores das gratificações aos dirigentes que estejam ligados à área industrial ou de produção de serviços devem ser informados nas contas de custos, respectivamente;
2) O valor de 13º salário pago a diretor contratado nos termos da Consolidação das Leis do Trabalho (CLT) é dedutível, desde que ele não esteja enquadrado no conceito de sócio, diretor ou administrador estabelecido no PN CST nº 48, de 1972.
3) As gratificações espontâneas devem ser adicionadas ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_31101070102,3.11.01.07.01.02,"(-) Ordenados, Salários, Gratificações e Outras Remunerações a Empregados","Contas que registram as despesas com ordenados, salários, gratificações e outras despesas com empregados, tais como: comissões, moradia, seguro de vida, contribuições pagas ao plano PAIT, despesas com programa de previdência privada, contribuições para os Fundos de Aposentadoria Programada Individual (Fali), e outras de caráter remuneratório.
Atenção:
1) As despesas correspondentes a salários, ordenados, gratificações e outras remunerações referentes à área de saúde, tais como assistência médica, odontológica e farmacêutica, devem ser indicadas na conta ""Assistência, médica, odontológica e farmacêutica a empregados"".
2) Não deve ser informado nesta linha o valor referente às participações dos empregados no lucro da pessoa jurídica. Esse valor deve ser informado na conta ""Participações de empregados"".
3) O valor das contribuições não compulsórias, destinadas a custear benefícios complementares assemelhados aos da previdência social, instituídos em favor dos empregados e dirigentes da pessoa jurídica, e para os Fundos de Aposentadoria Programada Individual (Fali) cujo ônus seja da pessoa jurídica, que exceder, no período de apuração, a vinte por cento do total dos salários dos empregados e da remuneração dos dirigentes da empresa, vinculados ao referido plano, deve ser adicionado ao Lucro Real.
4) As demais contribuições não compulsórias, exceto as destinadas a custear seguros e planos de saúde, devem ser adicionados ao Lucro Real"".",expense,,l10n_br_account_chart_template
account_template_31101070103,3.11.01.07.01.03,"(-) Outros Gastos com Pessoal","Contas que registrem os demais gastos com pessoal não especificados em contas anteriores.",expense,,l10n_br_account_chart_template
account_template_31101070104,3.11.01.07.01.04,"(-) Outros Serviços Prestados por Pessoa Física ou Jurídica","Contas que registram o valor das despesas correspondentes aos serviços prestados por:
1) Pessoa física: que não tenha vínculo empregatício com a pessoa jurídica declarante, tais como: comissões, corretagens, gratificações, honorários, direitos autorais e outras remunerações, inclusive as relativas a empreitadas de obras exclusivamente de trabalho e as decorrentes de fretes e carretos em geral.
2) Pessoa jurídica inclusive cooperativa de trabalho e locação de mão de obra;
Atenção: Somente são dedutíveis as despesas de comissões e corretagens quando, sobre elas, o credor tenha direito líquido e certo (PN CST nº 07, de 28 de janeiro de 1976).",expense,,l10n_br_account_chart_template
account_template_31101070105,3.11.01.07.01.05,"(-) Encargos Sociais - Previdência Social","Contas que registram as contribuições para a Previdência Social, não computadas nos custos (inclusive dos dirigentes - PN CST nº 35, de 31 de agosto de 1981).",expense,,l10n_br_account_chart_template
account_template_31101070106,3.11.01.07.01.06,"(-) Encargos Sociais - FGTS","Contas que registram as contribuições para o FGTS não computadas nos custos (inclusive dos dirigentes - PN CST nº 35, de 31 de agosto de 1981).",expense,,l10n_br_account_chart_template
account_template_31101070107,3.11.01.07.01.07,"(-) Encargos Sociais – Outros","Contas que registram os demais encargos sociais, não computadas nos custos ou nas contas Encargos Sociais -  Previdência Social ou Encargos Sociais -  FGTS.",expense,,l10n_br_account_chart_template
account_template_31101070108,3.11.01.07.01.08,"(-) Doações e Patrocínios de Caráter Cultural e Artístico (Lei no 8.313/1991)","Contas que registram as doações e patrocínios efetuados no período de apuração em favor de projetos culturais previamente aprovados pelo Ministério da Cultura ou pela Agência Nacional do Cinema (Ancine), observada a legislação de concessão dos projetos.
A pessoa jurídica que tiver efetuado doação ou patrocínio a projeto aprovado nos termos dos arts. 25 e 26 da Lei nº 8.313, de 23 de dezembro de 1991, ou nos termos desses dois artigos combinados com o § 6º do art. 39 da Medida Provisória nº 2.228-1, de 6 de agosto de 2001, cujos projetos são produzidos com os recursos de que trata o inciso X desse mesmo art. 39, pode deduzir o valor relativo às doações e/ou patrocínios como despesa operacional.
A pessoa jurídica que tiver efetuado doação ou patrocínio a projeto aprovado nos termos do art.18 da Lei nº 8.313, de 1991, com alterações promovidas pelo art. 1º da Lei nº 9.874, de 23 de novembro de 1999, e pelo art. 53 da MP nº 2.228-1, de 2001, com a redação dada pela Lei nº 10.454, de 2002, ou nos termos desses artigos combinados com o § 6º do art. 39 da Medida Provisória nº 2.228-1, de 6 de agosto de 2001, não pode efetuar qualquer dedução do valor correspondente às doações ou patrocínios como despesa operacional. Esse valor deve ser adicionado ao Lucro Real.
Atenção: Somente podem usufruir os benefícios fiscais referidos nesta linha os incentivadores que obedecerem, para suas doações ou patrocínios, o período definido pelas portarias editadas pelo MinC ou Ancine, publicadas no Diário Oficial da União, para homologação dos projetos beneficiários.",expense,,l10n_br_account_chart_template
account_template_31101070109,3.11.01.07.01.09,"(-) Doações de Aquisição de Vale-Cultura (Lei no 12.761/2012, art. 10)","Contas que registram o total do valor despendido no período de apuração a título de aquisição do vale-cultura.
O limite de dedução no percentual de um por cento será considerado isoladamente e não se submeterá a limite conjunto com outras deduções do imposto a título de incentivo. O valor excedente ao limite de dedução não poderá ser deduzido do imposto em períodos de apuração posteriores.
A pessoa jurídica beneficiária:
a) poderá deduzir o valor despendido a título de aquisição do vale-cultura como despesa operacional para fins de apuração do IRPJ; e
b) deverá adicionar o valor deduzido como despesa operacional, para fins de apuração da base de cálculo da CSLL.",expense,,l10n_br_account_chart_template
account_template_31101070110,3.11.01.07.01.10,"(-) Doações a Instituições de Ensino e Pesquisa (Lei nº 9.249/1995, art.13, § 2º)","Contas que registram as doações efetuadas às instituições de ensino e pesquisa cuja criação tenha sido autorizada por lei federal e que preencham os requisitos dos incisos I e II do art. 213 da Constituição Federal, de 1988, que são:
a) comprovação de finalidade não-lucrativa e aplicação dos excedentes financeiros em educação;
b) assegurar a destinação do seu patrimônio a outra escola comunitária, filantrópica ou confessional, ou ao Poder Público, no caso de encerramento de suas atividades.
A sua dedutibilidade está limitada a 1,5% (um e meio por cento) do lucro operacional, antes de computada esta dedução e a das doações a entidades civis.",expense,,l10n_br_account_chart_template
account_template_31101070111,3.11.01.07.01.11,"(-) Doações a Entidades Civis","Contas que registram as doações efetuadas a:
a) entidades civis, legalmente constituídas no Brasil, sem fins lucrativos, que prestem serviços gratuitos em benefício de empregados da pessoa jurídica doadora, e respectivos dependentes, ou em benefício da comunidade na qual atuem; e
b) Organizações da Sociedade Civil de Interesse Público (OSCIP), qualificadas segundo as normas estabelecidas na Lei nº 9.790, de 23 de março de 1999.
Para fins de dedução na apuração do lucro real, as referidas doações estão limitadas a 2% (dois por cento) do lucro operacional da pessoa jurídica, antes de computadas essas deduções, observadas as seguintes regras:
a) as doações, quando em dinheiro, devem ser feitas mediante crédito em conta corrente bancária diretamente em nome da entidade beneficiária;
b) a pessoa jurídica doadora deve manter em arquivo, à disposição da fiscalização, declaração, segundo modelo aprovado pela IN SRF nº 87, de 31 de dezembro de 1996, fornecida pela entidade beneficiária, em que está se compromete a aplicar integralmente os recursos recebidos na realização de seus objetivos sociais, com identificação da pessoa física responsável pelo seu cumprimento, e a não distribuir lucros, bonificações ou vantagens a dirigentes, mantenedores ou associados, sob nenhuma forma ou pretexto (Lei nº 9.249, de 1995, art. 13, § 2º, inciso III, alínea b);
Atenção:
1) A condição estabelecida no item b não alcança a hipótese de remuneração de dirigente em decorrência de vínculo empregatício, pelas Organizações da Sociedade Civil de Interesse Público (OSCIP), qualificadas segundo as normas estabelecidas na Lei nº 9.790, de 1999, e pelas Organizações Sociais (OS), qualificadas consoante os dispositivos da Lei nº 9.637, de 15 de maio de 1998.
2) O disposto no item anterior aplica-se somente à remuneração não superior, em seu valor bruto, ao limite estabelecido para a remuneração de servidores do Poder Executivo Federal.
c) a entidade civil beneficiária deve ser reconhecida de utilidade pública por ato formal de órgão competente da União.
Atenção: O disposto neste item não se aplica às OSCIP.
d) a dedutibilidade fica condicionada a que a entidade beneficiária tenha sua condição de utilidade pública ou de OSCIP renovada anualmente pelo órgão competente da União, mediante ato formal.
Atenção: Essa renovação:
a) somente será concedida a entidade que comprove, perante o órgão competente da União, ter cumprido, no ano-calendário anterior ao do pedido, todas as exigências e condições estabelecidas;
b) produzirá efeitos para o ano-calendário subsequente ao de sua formalização.
O valor que exceder o limite permitido deve ser adicionado ao Lucro Real. ",expense,,l10n_br_account_chart_template
account_template_31101070112,3.11.01.07.01.12,"(-) Outras Contribuições, Doações e Patrocínios","Contas que registram as doações feitas, entre outras, aos Fundos controlados pelos Conselhos Municipais, Estaduais e Nacional dos Direitos da Criança e do Adolescente e Atividades de Caráter Desportivo. O valor dessas doações aos Fundos dos Direitos da Criança e do Adolescente e Atividade de Caráter Desportivo não é dedutível como despesa operacional na determinação do lucro real e da base de cálculo da contribuição social sobre o lucro líquido, mas pode ser deduzido diretamente do imposto devido.
O valor indicado nesta linha deve, também, ser adicionado ao Lucro Real.
Atenção:
1) Os valores das doações e patrocínios de caráter cultural e artístico, das doações a instituições de ensino e pesquisa e das doações a entidades civis (Lei nº 9.249, de 1995, art. 13, § 2º), devem ser indicados nas respectivas contas.
2) O valor da contribuição sindical deve ser informado na conta de ""Outras Despesas Operacionais"".",expense,,l10n_br_account_chart_template
account_template_31101070113,3.11.01.07.01.13,"(-) Alimentação do Trabalhador","Contas que registram o valor das despesas com alimentação do pessoal não ligado à produção, realizadas durante o período de apuração, ainda que a pessoa jurídica não tenha Programa de Alimentação do Trabalhador aprovado pelo Ministério do Trabalho. As despesas correspondentes, inclusive com cestas básicas de alimentos, somente podem ser dedutíveis quando a pessoa jurídica fornecer alimentação, indistintamente, a todos os seus empregados. ",expense,,l10n_br_account_chart_template
account_template_31101070114,3.11.01.07.01.14,"(-) PIS/PASEP","Contas que registram a parcela das Contribuições para o PIS/PASEP incidente sobre as demais receitas operacionais.",expense,,l10n_br_account_chart_template
account_template_31101070115,3.11.01.07.01.15,"(-) COFINS","Contas que registram a parcela da COFINS incidente sobre as demais receitas operacionais",expense,,l10n_br_account_chart_template
account_template_31101070116,3.11.01.07.01.16,"(-) Demais Impostos, Taxas e Contribuições, exceto IR e CSLL","Contas que registram os demais tributos e contribuições. Os valores indicados nesta conta são dedutíveis, para efeito de determinação do lucro real, no período de apuração em que ocorrer o fato gerador.
Não devem ser incluídas as importâncias:
a) incorporadas ao custo de bens do ativo não-circulante, exceto realizável a longo prazo;
b) correspondentes aos impostos não recuperáveis, incorporados ao custo das matérias-primas, materiais secundários, materiais de embalagem e mercadorias destinadas à revenda;
c) correspondentes aos impostos recuperáveis;
d) correspondentes aos impostos e contribuições redutores da receita bruta;
e) correspondentes às Contribuições para o PIS/PASEP e à COFINS incidentes sobre as demais receitas operacionais;
f) correspondentes à contribuição social sobre o lucro líquido e ao imposto de renda devidos. ",expense,,l10n_br_account_chart_template
account_template_31101070117,3.11.01.07.01.17,"(-) Arrendamento Mercantil","Contas que registram as despesas, não computadas nos custos, pagas ou creditadas a título de contraprestação de arrendamento mercantil, decorrentes de contrato celebrado com observância da Lei nº 6.099, de 12 de setembro de 1974, com as alterações da Lei nº 7.132, de 26 de outubro de 1983, e da Portaria MF nº 140, de 1984.
Atenção: As despesas relativas ao arrendamento de bens que não sejam intrinsecamente vinculados com a comercialização de bens ou serviços devem ser adicionadas ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_31101070118,3.11.01.07.01.18,"(-) Aluguéis","Contas que registram as despesas com aluguéis não decorrentes de arrendamento mercantil.
Atenção: As despesas relativas a aluguéis de bens móveis ou imóveis que não sejam intrinsecamente relacionados com a comercialização dos bens ou serviços devem ser adicionadas ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_31101070119,3.11.01.07.01.19,"(-) Despesas com Veículos e de Conservação de Bens e Instalações","Contas que registram as despesas relativas aos bens que não estejam ligados diretamente à produção, as realizadas com reparos que não impliquem aumento superior a um ano da vida útil do bem, prevista no ato de sua aquisição, e as relativas a combustíveis e lubrificantes para veículos.
Atenção: Somente são dedutíveis, para fins de apuração do lucro real, as despesas com Veículos e de Conservação de Bens e Instalações relativas a bens intrinsecamente vinculados com a comercialização de bens ou serviços. Os gastos considerados indedutíveis devem ser adicionados ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_31101070120,3.11.01.07.01.20,"(-) Propaganda, Publicidade e Patrocínio","Contas que registram as despesas com propaganda, publicidade e patrocínio.
Atenção: Essas despesas são dedutíveis nas condições estabelecidas no art. 366 do Decreto nº 3.000, de 1999, segundo o regime de competência.",expense,,l10n_br_account_chart_template
account_template_31101070121,3.11.01.07.01.21,"(-) Propaganda, Publicidade e Patrocínio de Assoc. Desportivas que Mantenha Equipe de Futebol Profissional","Contas que registram as despesas com propaganda e publicidade e patrocínios destinados a manutenção de Equipes de Futebol profissional.",expense,,l10n_br_account_chart_template
account_template_31101070122,3.11.01.07.01.22,"(-) Multas","Contas que registram as despesas com multas.                                                                                                                                                                                                                                                                         São totalmente indedutíveis não só as multas impostas por infrações fiscais de que resulte falta ou insuficiência de pagamento de tributo ou contribuição, como também aquelas que decorram de infrações a normas não tributárias (multas de trânsito, por exemplo). São dedutíveis as multas fiscais de natureza compensatória e aquelas impostas por descumprimento de obrigações tributárias, meramente acessórias, de que não resulte falta ou insuficiência de pagamento de tributo ou contribuição (PN CST nº 61, de 1979).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 Atenção: Os valores das multas indedutíveis devem ser adicionados ao Lucro Real.                                                      ",expense,,l10n_br_account_chart_template
account_template_31101070123,3.11.01.07.01.23,"(-) Encargos de Depreciação","Contas que registram o valor de depreciação com bens, inclusive bens adquiridos sob a modalidade de arrendamento financeiro, não aplicados diretamente na produção.
Atenção: Somente são dedutíveis, para fins de apuração do lucro real, as despesas com Encargos de Depreciação de Bens e Instalações intrinsecamente vinculados com a comercialização de bens ou serviços. Os gastos considerados indedutíveis devem ser adicionados ao Lucro Real.",expense_depreciation,,l10n_br_account_chart_template
account_template_31101070124,3.11.01.07.01.24,"(-) Encargos de Amortização","Contas que registram o valor de amortização de direitos ou bens intangíveis, inclusive objeto de arrendamento financeiro, não aplicados diretamente na produção.
Atenção: Somente são dedutíveis, para fins de apuração do lucro real, as despesas com Encargos de Amortização de Bens e Instalações intrinsecamente vinculados com a comercialização de bens ou serviços. Os gastos considerados indedutíveis devem ser adicionados ao Lucro Real.",expense_depreciation,,l10n_br_account_chart_template
account_template_31101070125,3.11.01.07.01.25,"(-) Perdas em Operações de Crédito","Contas que registram as perdas efetivas no recebimento de créditos decorrentes das atividades da pessoa jurídica.",expense,,l10n_br_account_chart_template
account_template_31101070126,3.11.01.07.01.26,"(-) Provisões para Férias","Contas que registram as despesas com a constituição de provisão para o pagamento de remuneração correspondente a férias e adicional de férias de empregados, inclusive encargos sociais (Decreto nº 3.000, de 1999, art. 337, e PN CST nº 7, de 1980).",expense,,l10n_br_account_chart_template
account_template_31101070127,3.11.01.07.01.27,"(-) Provisões para 13º Salário de Empregados","Contas que registram as despesas com a constituição de provisão para 13º salário, no caso de apuração trimestral do imposto, inclusive encargos sociais (Decreto nº 3.000, de 1999, art. 338).",expense,,l10n_br_account_chart_template
account_template_31101070128,3.11.01.07.01.28,"(-) Provisão para Perda de Estoque","Contas que registram as despesas com a constituição de provisão para perda de estoque.  As pessoas jurídicas que exerçam as atividades de editor (a pessoa física ou jurídica que adquire o direito de reprodução de livros, dando a eles tratamento adequado à leitura), distribuidor (a pessoa jurídica que opera no ramo de compra e venda de livros por atacado) e livreiro (a pessoa jurídica ou representante comercial autônomo que se dedica à venda de livros), poderão indicar nesta linha, a provisão para perda de estoques, calculada no último dia de cada período de apuração do imposto de renda e da contribuição social sobre o lucro líquido, correspondente a 1/3 (um terço) do valor do estoque existente naquela data, na forma da IN SRF nº 412, de 23 de março de 2004. Ao fim de cada exercício financeiro legal será feito o ajustamento da provisão dos respectivos estoques.",expense,,l10n_br_account_chart_template
account_template_31101070129,3.11.01.07.01.29,"(-) Demais Provisões","Contas que registram às despesas com provisões não relacionadas nas linhas anteriores, constituídas no decorrer do período de apuração.
Atenção: Os valores indicados nesta linha são totalmente indedutíveis, devendo ser adicionados ao Lucro Real",expense,,l10n_br_account_chart_template
account_template_31101070130,3.11.01.07.01.30,"(-) Gratificações a Administradores","Contas que registram as gratificações a administradores.
Os pagamentos e créditos a esse título são totalmente indedutíveis. Por isso, seu montante deve ser adicionado ao Lucro Real.",expense,,l10n_br_account_chart_template
account_template_31101070131,3.11.01.07.01.31,"(-) Royalties e Assistência Técnica - no PAÍS","Contas que registram as despesas correspondentes às importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no Brasil, a título de royalties e assistência técnica, científica ou assemelhada, que não estejam relacionados com a produção de bens e/ou serviços.",expense,,l10n_br_account_chart_template
account_template_31101070132,3.11.01.07.01.32,"(-) Royalties e Assistência Técnica - no EXTERIOR","Contas que registram as despesas correspondentes às importâncias pagas a beneficiário pessoa física ou jurídica, residente ou domiciliado no exterior, a título de royalties e assistência técnica, científica ou assemelhada, que não estejam relacionados com a produção de bens e/ou serviços.",expense,,l10n_br_account_chart_template
account_template_31101070133,3.11.01.07.01.33,"(-) Assistência Médica, Odontológica e Farmacêutica a Empregados","Contas que registram as despesas com assistência médica, odontológica e farmacêutica.
Atenção: O valor referente à contratação de serviços de profissionais liberais sem vínculo empregatício ou de sociedades civis deve ser informado nas contas Prestação de Serviços por Pessoa Física sem Vínculo Empregatício ou Prestação de Serviço Pessoa Jurídica, conforme o caso.",expense,,l10n_br_account_chart_template
account_template_31101070134,3.11.01.07.01.34,"(-) Pesquisas Científicas e Tecnológicas","Contas que registram as despesas efetuadas a esse título, inclusive a contrapartida das amortizações daquelas registradas no ativo diferido",expense,,l10n_br_account_chart_template
account_template_31101070135,3.11.01.07.01.35,"(-) Bens de Pequeno Valor Unitário ou de Vida Útil de até um Ano Deduzidos como Despesa","Contas que registram o valor de aquisição de bens do ativo imobilizado cujo prazo de vida útil não ultrapasse um ano, ou, caso exceda esse prazo, tenha valor unitário igual ou inferior a R$ 1 200,00 (mil duzentos reais) (Lei nº 12.973, de 2014, art. 15).",expense,,l10n_br_account_chart_template
account_template_31101070136,3.11.01.07.01.36,"(-) Despesas com Energia Elétrica","Contas que registram as despesas com energia elétrica.",expense,,l10n_br_account_chart_template
account_template_31101070137,3.11.01.07.01.37,"(-) Despesas com Água e Esgoto","Contas que registram as despesas com água e esgoto.",expense,,l10n_br_account_chart_template
account_template_31101070138,3.11.01.07.01.38,"(-) Despesas com Telefone e Internet","Contas que registram as despesas com telefone e internet.",expense,,l10n_br_account_chart_template
account_template_31101070139,3.11.01.07.01.39,"(-) Despesas com Correios e Malotes","Contas que registram as despesas com correios e malotes.",expense,,l10n_br_account_chart_template
account_template_31101070140,3.11.01.07.01.40,"(-) Despesas com Seguros","Contas que registram as despesas com seguros.",expense,,l10n_br_account_chart_template
account_template_31101090101,3.11.01.09.01.01,"(-) Variações Cambiais Passivas","Contas que registram as perdas monetárias passivas resultantes da atualização dos direitos de créditos e das obrigações, calculadas com base nas variações nas taxas de câmbio (Lei nº 9.069, de 1995, art. 52, e Lei nº 9.249, de 1995, art. 8º).
Incluir, nesta linha, a variação cambial passiva correspondente:
a) à atualização das obrigações e dos créditos em moeda estrangeira, registrada em qualquer data e apurada no encerramento do período de apuração em função da taxa de câmbio vigente;
b) às operações com moeda estrangeira e conversão de obrigações para moeda nacional, ou novação dessas obrigações, ou sua extinção, total ou parcial, em virtude de capitalização, dação em pagamento, compensação, ou qualquer outro modo, desde que observadas as condições fixadas pelo Banco Central do Brasil.
Atenção: As variações cambiais passivas decorrentes dos direitos de crédito e de obrigações, em função da taxa de câmbio, são consideradas como despesa financeira, inclusive para fins de cálculo do lucro da exploração (Lei nº 9.718, art. 9º c/c art. 17).",expense,,l10n_br_account_chart_template
account_template_31101090102,3.11.01.09.01.02,"(-) Perdas Incorridas no Mercado de Renda Variável, exceto Day-Trade","Contas que registram:
a) o somatório das perdas incorridas, em cada mês do período de apuração, em operações realizadas nas bolsas de valores, de mercadorias, de futuros e assemelhadas, existentes no País;
b) as perdas incorridas nas alienações, fora de bolsa, de ouro, ativo financeiro, e de participações societárias, exceto as alienações de participações societárias permanentes em sociedades coligadas e controladas e de participações societárias que permanecerem no ativo da pessoa jurídica até o término do ano-calendário seguinte ao de suas aquisições; e
c) as perdas em operações de swap e no resgate de quota de fundo de investimento que mantenha, no mínimo, 67% (sessenta e sete por cento) de ações negociadas no mercado à vista de bolsa de valores ou entidade assemelhada (Lei nº 9.532, de 1997, art. 28, alterado pela MP nº 1.636, de 1998, art. 2º, e reedições).
São consideradas assemelhadas às bolsas de valores, de mercadorias e de futuros as entidades cujo objeto social seja análogo ao das referidas bolsas e que funcionem sob a supervisão e fiscalização da Comissão de Valores Mobiliários (CVM). Atenção:
1) As perdas apuradas nas operações de renda variável, exceto day-trade, somente são dedutíveis na determinação do lucro real até o limite dos ganhos auferidos em operações de mesma natureza, observado o disposto nos itens 3 a 5. As perdas não deduzidas em um período de apuração podem sê-lo nos períodos de apuração subsequentes. A parcela não dedutível no período de apuração deve ser controlada na Parte B do Lalur.
2) A partir de 1º de janeiro de 2000, as perdas apuradas em operações, exceto day-trade, no mercado à vista de ações somente são compensadas com os ganhos líquidos auferidos em operações, exceto day-trade, realizadas exclusivamente nesse mercado.
3) O saldo de perdas decorrentes de operações, exceto day-trade, existente em 31 de dezembro de 1999 pode ser compensado com os ganhos líquidos auferidos:
a) no mercado à vista de ações, se as perdas decorreram de operações, exceto day-trade, realizadas exclusivamente nesse mercado; e
b) em quaisquer mercados, se as perdas decorreram de operações, exceto day-trade, realizadas em mercados diversificados.
4) As limitações de realização de perdas, de que tratam as instruções de preenchimento desta linha, não se aplicam às pessoas jurídicas citadas no inciso I do art. 35 da IN SRF nº 25, de 6 de março de 2001, e às operações de swap utilizadas como cobertura (hedge).",expense,,l10n_br_account_chart_template
account_template_31101090103,3.11.01.09.01.03,"(-) Perdas em Operações Day-Trade","Contas que registram o somatório das perdas diárias apuradas, em cada mês do período de apuração, em operações day-trade.
Não se caracteriza como day-trade o exercício da opção e a venda ou compra do ativo no mercado à vista, no mesmo dia.
Também não se caracterizam como day-trade as operações iniciadas por intermédio de uma instituição e encerradas em outra, quando houver a liquidação física mediante movimentação de títulos ou valores mobiliários em custódia",expense,,l10n_br_account_chart_template
account_template_31101090104,3.11.01.09.01.04,"(-) Despesas de Juros sobre o Capital Próprio","Contas que registram os juros pagos ou creditados individualizadamente a titular, sócios ou acionistas, a título de remuneração do capital próprio, calculados sobre as contas do patrimônio líquido e limitados à variação, pro rata dia, da Taxa de Juros de Longo Prazo (TJLP), observando-se o regime de competência (Lei nº 9.249, de 1995, art. 9º).                                                                                                                                                                                                                                                                                                                                           Atenção: Quanto à dedutibilidade dos juros como despesa operacional, para fins de determinação do lucro real e da base de cálculo da CSLL.",expense,,l10n_br_account_chart_template
account_template_31101090105,3.11.01.09.01.05,"(-) Despesas de Remuneração de Debêntures","Contas que registram as despesas de Remuneração de Debêntures.",expense,,l10n_br_account_chart_template
account_template_31101090106,3.11.01.09.01.06,"(-) Juros com Empréstimos de Pessoas Vinculadas ou Situadas em País com Tributação favorecida","Contas que registram os juros pagos ou creditados por fonte situada no Brasil à pessoa física ou jurídica, vinculada nos termos do art. 23 da Lei nº 9.430, de 27 de dezembro de 1996, residente ou domiciliada no exterior, não constituída em país ou dependência com tributação favorecida ou sob regime fiscal privilegiado, observado o art. 24 da Lei nº 12.249, de 11 de junho de 2010.
Indicar também, os juros pagos ou creditados por fonte situada no Brasil à pessoa física ou jurídica residente, domiciliada ou constituída no exterior, em país ou dependência com tributação favorecida ou sob regime fiscal privilegiado, nos termos dos arts. 24 e 24-A da Lei nº 9.430, de 27 de dezembro de 1996, observado o art. 25 da Lei nº 12.249, de 2010.",expense,,l10n_br_account_chart_template
account_template_31101090107,3.11.01.09.01.07,"(-) Despesas Financeiras Relativas a Arrendamento","Contas que registram a contrapartida da realização do ajuste ao valor presente dos elementos monetários do passivo decorrentes de operações de longo prazo ou quando houver efeito relevante relativos a arrendamento.",expense,,l10n_br_account_chart_template
account_template_31101090108,3.11.01.09.01.08,"(-) Outras Despesas Financeiras","Contas que registram as despesas relativas a juros, não incluídas nas contas específicas, tais despesas serão obrigatoriamente apropriadas, segundo o regime de competência.
Atenção:
1) As variações monetárias passivas decorrentes da atualização das obrigações, em função de índices ou coeficientes aplicáveis por disposição legal ou contratual, devem ser informadas como despesa financeira.
2) As variações cambiais passivas não devem ser informadas nesta linha, e sim em conta específica.",expense,,l10n_br_account_chart_template
account_template_31101090109,3.11.01.09.01.09,"(-) Resultados Negativos em Participações Societárias Avaliadas pelo Método de Equivalência Patrimonial","Contas que registram as perdas por ajustes no valor de investimentos relevantes avaliados pelo método da equivalência patrimonial, decorrentes de prejuízos apurados nas controladas e coligadas. O valor indicado deve ser adicionado ao lucro líquido, para determinação do lucro real.
Atenção:
1) Considera-se controlada a filial, a agência, a sucursal, a dependência ou o escritório de representação no exterior, sempre que os respectivos ativos e passivos não estejam incluídos na contabilidade da investidora, por força de normatização específica.
2) Devem, também, ser indicados nesta linha os resultados negativos derivados de participações societárias no exterior, avaliadas pelo patrimônio líquido. Incluem-se, nestas informações, as perdas apuradas em filiais, sucursais e agências da pessoa jurídica localizadas no exterior.",expense,,l10n_br_account_chart_template
account_template_31101090110,3.11.01.09.01.10,"(-) Resultados Negativos em SCP Avaliadas pelo Método de Equivalência Patrimonial","Conta utilizada pelos sócios ostensivos, pessoas jurídicas, de sociedades em conta de participação, para indicar as perdas por ajustes no valor de participação em SCP, avaliada pelo método da equivalência patrimonial. O valor dessas perdas deve ser adicionado ao lucro líquido na determinação do lucro real ",expense,,l10n_br_account_chart_template
account_template_31101090111,3.11.01.09.01.11,"(-) Perdas em Operações Realizadas no Exterior","Contas que registram as perdas em operações realizadas no exterior diretamente pela pessoa jurídica domiciliada no Brasil, com exceção das perdas de capital decorrentes da alienação de bens e direitos do ativo permanente situados no exterior, que devem ser indicadas em conta específica. O valor aqui indicado deve ser adicionado ao lucro líquido para fins de apuração do lucro real",expense,,l10n_br_account_chart_template
account_template_31101090112,3.11.01.09.01.12,"(-) Perdas Estimadas Decorrentes de Teste de Recuperabilidade (Impairment)","Contas que registram a redução dos valores registrados no ativo decorrentes de análise sobre a recuperação dos ativos (teste de recuperabilidade).",expense,,l10n_br_account_chart_template
account_template_31101090113,3.11.01.09.01.13,"(-) Despesas de Reclassificação de Ajustes de Avaliação Patrimonial ","Contas que registram as contrapartidas da realização do grupo Ajustes de Avaliação Patrimonial do PL reconhecidas no resultado do exercício em obediência ao regime de competência.",expense,,l10n_br_account_chart_template
account_template_31101090114,3.11.01.09.01.14,"(-) Despesas de Reclassificação de Ajustes de Avaliação Patrimonial -Reflexo","Contas que registram as contrapartidas da realização do grupo Ajustes de Avaliação Patrimonial do PL- reflexos reconhecidas no resultado do exercício em obediência ao regime de competência.",expense,,l10n_br_account_chart_template
account_template_31101090115,3.11.01.09.01.15,"(-) Despesas Financeiras Decorrentes dos Ajustes ao Valor Presente","Contas que registram os encargos de depreciação de bens objeto de arrendamento.",expense_depreciation,,l10n_br_account_chart_template
account_template_31101090116,3.11.01.09.01.16,"(-) Encargos de Depreciação de Bens Objeto de Arrendamento","Contas que registram os encargos de depreciação de bens objeto de leasing financeiro.",expense_depreciation,,l10n_br_account_chart_template
account_template_31101090117,3.11.01.09.01.17,"(-) Encargos de Amortização de Mais - Valia","Contas que registram os encargos de amortização de mais-valia.",expense_depreciation,,l10n_br_account_chart_template
account_template_31101090118,3.11.01.09.01.18,"(-) Aluguéis de Bens Imóveis- Locador  Parte Relacionada","Contas que registram os aluguéis de bens imóveis a parte relacionada",expense,,l10n_br_account_chart_template
account_template_31101090119,3.11.01.09.01.19,"(-) Aluguéis de Bens Imóveis Locador  Parte Não Relacionada","Contas que registram os aluguéis de bens imóveis a parte não relacionada",expense,,l10n_br_account_chart_template
account_template_31101090120,3.11.01.09.01.20,"(-) Despesas com Empréstimos de Valores Mobiliários","Contas que registram as despesas com empréstimos de valores mobiliários.",expense,,l10n_br_account_chart_template
account_template_31101090121,3.11.01.09.01.21,"(-) Despesas com Corretagem e Emolumentos","Contas que registram as despesas com corretagem e emolumentos.",expense,,l10n_br_account_chart_template
account_template_31101090122,3.11.01.09.01.22,"(-) Despesas  com Deságio na Cessão de Títulos","Contas que registram as despesas com deságio na cessão de títulos decorrentes de securitização",expense,,l10n_br_account_chart_template
account_template_31101090123,3.11.01.09.01.23,"(-) Despesas Incorridas em Operações de Mútuo – Parte Relacionada","Contas que registram os juros incorridos em operações de mútuo com partes relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",expense,,l10n_br_account_chart_template
account_template_31101090124,3.11.01.09.01.24,"(-) Despesas Incorridas em Operações de Mútuo – Parte Não Relacionada","Contas que registram os juros incorridos em operações de mútuo com partes não relacionadas com a declarante, conforme conceito definido no CPC 05(R1), itens 09 a 12.",expense,,l10n_br_account_chart_template
account_template_31101090125,3.11.01.09.01.25,"(-) Despesas Incorridas  em Outros Passivos Financeiros Mensurados Pelo Custo Amortizado","Contas que registram os juros auferidos com passivos financeiros mensurados pelo custo amortizado.",expense,,l10n_br_account_chart_template
account_template_31101090126,3.11.01.09.01.26,"(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros para Negociação - Não Hedge - Valor Justo pelo Resultado","Contas que registram as perdas decorrentes de ajustes a valor justo de instrumentos financeiros para negociação, exceto hedge, avaliados a valor justo pelo resultado.",expense,,l10n_br_account_chart_template
account_template_31101090127,3.11.01.09.01.27,"(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros Disponíveis para Venda - Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram as perdas decorrentes de ajustes a valor justo de instrumentos financeiros disponíveis para venda no momento da reclassificação dos ajustes de avaliação patrimonial.",expense,,l10n_br_account_chart_template
account_template_31101090128,3.11.01.09.01.28,"(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros de Hedge de Valor Justo","Contas que registram as perdas decorrentes de ajustes a valor justo de instrumentos financeiros de hedge.",expense,,l10n_br_account_chart_template
account_template_31101090129,3.11.01.09.01.29,"(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros de Hedge -  Reclassificação de Ajustes de Avaliação Patrimonial","Contas que registram as perdas decorrentes de ajustes a valor justo de instrumentos financeiros de hedge no momento da reclassificação dos ajustes de avaliação patrimonial.",expense,,l10n_br_account_chart_template
account_template_31101090130,3.11.01.09.01.30,"(-) Perda de Ajuste a Valor Justo - Item Objeto de Hedge de Valor Justo","Contas que registram as perdas decorrentes de ajustes a valor justo de item objeto de hedge.",expense,,l10n_br_account_chart_template
account_template_31101090131,3.11.01.09.01.31,"(-) Perda de Ajuste a Valor Justo - Propriedade para Investimento","Contas que registram as perdas decorrentes de ajustes a valor justo de propriedades para investimento.",expense,,l10n_br_account_chart_template
account_template_31101090132,3.11.01.09.01.32,"(-) Perda de Ajuste a Valor Justo - Ativo Biológico Consumível","Contas que registram as perdas decorrentes de ajustes a valor justo de ativo biológico consumível.",expense,,l10n_br_account_chart_template
account_template_31101090133,3.11.01.09.01.33,"(-) Perda de Ajuste a Valor Justo - Ativo Biológico de Produção","Contas que registram as perdas decorrentes de ajustes a valor justo de ativo biológico de produção.",expense,,l10n_br_account_chart_template
account_template_31101090134,3.11.01.09.01.34,"(-) Perda de Ajuste a Valor Justo - Ativos Não Circulantes Mantidos para Venda","Contas que registram as perdas decorrentes de ajustes a valor justo de ativos não circulantes mantidos para venda.",expense,,l10n_br_account_chart_template
account_template_31101090135,3.11.01.09.01.35,"(-) Perda de Ajuste a Valor Justo - Subscrição de Capital com demais Bens","Contas que registram as perdas decorrentes de ajustes a valor justo de subscrição de capital com demais bens.",expense,,l10n_br_account_chart_template
account_template_31101090136,3.11.01.09.01.36,"(-) Perda de Ajuste a Valor Justo - Subscrição de Capital com Participação Societária","Contas que registram as perdas decorrentes de ajustes a valor justo de subscrição de capital com participação societária.",expense,,l10n_br_account_chart_template
account_template_31101090137,3.11.01.09.01.37,"(-) Perda de Ajuste a Valor Justo - Aquisição de Participação Societária em Estágios","Contas que registram as perdas decorrentes de ajustes a valor justo de aquisição de participação societária em estágios.",expense,,l10n_br_account_chart_template
account_template_31101090138,3.11.01.09.01.38,"(-) Perda de Ajuste a Valor Justo - Decorrente de Permuta de Ativos ou Passivos","Contas que registram as perdas decorrentes de ajustes a valor justo devido a permuta de ativos ou passivos.",expense,,l10n_br_account_chart_template
account_template_31101090139,3.11.01.09.01.39,"(-) Perda de Ajuste a Valor Justo - Outras Operações","Contas que registram as perdas decorrentes de ajustes a valor justo de outras operações não classificáveis neste plano de contas.",expense,,l10n_br_account_chart_template
account_template_31101090199,3.11.01.09.01.99,"(-) Outras Despesas Operacionais","Contas que registram as demais despesas que, por definição legal, sejam consideradas operacionais, não enquadráveis contas específicas.",expense,,l10n_br_account_chart_template
account_template_31101110101,3.11.01.11.01.01,"Receitas na Alienação de Bens Integrantes do Ativo Circulante ou do Ativo Realizável a Longo Prazo","Contas que registram as receitas auferidas quando da conclusão da alienação dos ativos destinados a vendas.  Esses ativos ou grupo de ativos foram reclassificados para o circulante quando da decisão da administração da companhia na venda dos ativos ou grupo de ativos.",income,,l10n_br_account_chart_template
account_template_31101110102,3.11.01.11.01.02,"Receitas de Alienações de Bens do Ativo Não Circulante","Contas que registram as receitas auferidas por meio de alienações, inclusive por desapropriação de bens e direitos classificados em investimentos, imobilizado e intangível.",income,,l10n_br_account_chart_template
account_template_31101110104,3.11.01.11.01.04,"(-) Valor Contábil de Bens Integrantes do Ativo Circulante ou do Ativo Realizável a Longo Prazo Alienados","Contas que registram o valor contábil quando da conclusão da alienação dos ativos destinados a vendas.  Esses ativos ou grupo de ativos foram reclassificados para o circulante quando da decisão da administração da companhia na venda dos ativos ou grupo de ativos.",expense,,l10n_br_account_chart_template
account_template_31101110105,3.11.01.11.01.05,"(-) Valor Contábil dos Bens do Ativo Não Circulante Alienados","Contas que registram o valor contábil por meio de alienações, inclusive por desapropriação de bens e direitos classificados em investimentos, imobilizado e intangível.",expense,,l10n_br_account_chart_template
account_template_31105010102,3.11.05.01.01.02,"(-) Contribuições para Assistência ou Previdência de Empregados","Contas que registram as contribuições para instituições ou fundos de assistência ou previdência de empregados, baseadas nos lucros. Para efeito do imposto de renda, essas contribuições somente podem ser deduzidas quando pagas a entidades de previdência privada expressamente autorizadas a funcionar. As contribuições que não satisfaçam as condições legais devem ser adicionadas ao Lucro Real. Não indicar, nesta linha, aquelas contribuições já deduzidas como custo ou despesa operacional.",expense,,l10n_br_account_chart_template
account_template_31105010199,3.11.05.01.01.99,"(-) Outras Participações de Empregados","Contas que registram as demais participações de empregados.",expense,,l10n_br_account_chart_template
account_template_31105010301,3.11.05.01.03.01,"(-) Participações de Administradores e Partes Beneficiárias","Contas que registram as participações nos lucros atribuídas a administradores, sócio, titular de empresa individual e a portadores de partes beneficiárias, durante o período de apuração.",expense,,l10n_br_account_chart_template
account_template_31105010302,3.11.05.01.03.02,"(-) Participações de Debêntures","Contas que registram as participações nos lucros da companhia atribuídas a debêntures de sua emissão.",expense,,l10n_br_account_chart_template
account_template_31105010399,3.11.05.01.03.99,"(-) Outras Participações","Contas que registram as outras participações não especificadas anteriormente.",expense,,l10n_br_account_chart_template
account_template_31201010101,3.12.01.01.01.01,"Contribuição Social sobre o Lucro Líquido (Atividade Rural)","Contas que registram a soma das provisões para a CSLL calculadas sobre a base de cálculo correspondente ao período de apuração. A sua constituição é obrigatória para todas as pessoas jurídicas tributadas com base no lucro real.
As cooperativas devem informar, nesta linha, a provisão da CSLL sobre os resultados das operações realizadas com os não-associados",expense,,l10n_br_account_chart_template
account_template_31201010111,3.12.01.01.01.11,"Contribuição Social sobre o Lucro Líquido - Lucros Diferidos (Atividade Rural)","Contas que registram a soma das provisões para a CSLL calculadas sobre os lucros diferidos da atividade rural, se for o caso. A sua constituição é obrigatória para todas as pessoas jurídicas tributadas com base no lucro real.
As cooperativas devem informar, nesta linha, a provisão da CSLL sobre os resultados das operações realizadas com os não-associados",expense,,l10n_br_account_chart_template

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="br_3_01_01_05_01_47" model="account.account.template">
        <field name="code">3.01.01.05.01.47</field>
        <field name="name">Ganho Cambial</field>
        <field name="account_type">income_other</field>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
    </record>

    <record id="br_3_11_01_09_01_40" model="account.account.template">
        <field name="code">3.11.01.09.01.40</field>
        <field name="name">Perda Cambial</field>
        <field name="account_type">expense</field>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
    </record>

    <record id="l10n_br_account_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="account_template_101010401"/>
        <field name="property_account_payable_id" ref="account_template_201010301"/>
        <field name="property_account_expense_categ_id" ref="account_template_30101030101"/>
        <field name="property_account_income_categ_id" ref="account_template_30101010105"/>
        <field name="property_tax_payable_account_id" ref="account_template_202011003"/>
        <field name="property_tax_receivable_account_id" ref="account_template_102010802"/>
        <field name="income_currency_exchange_account_id" ref="br_3_01_01_05_01_47"/>
        <field name="expense_currency_exchange_account_id" ref="br_3_11_01_09_01_40"/>
        <field name="default_pos_receivable_account_id" ref="account_template_101010402"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="account_template_31101010202"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="account_template_30101050148"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Sales within the same state -->
    <record id="account_fiscal_position_same_state_sale1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_1"/>
        <field name="tax_src_id" ref="tax_template_out_icms_externo17"/>
        <field name="tax_dest_id" ref="tax_template_out_icms_interno17"/>
    </record>

    <!-- Purchases within the same state -->
    <record id="account_fiscal_position_same_state_purchase1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_1"/>
        <field name="tax_src_id" ref="tax_template_in_icms_externo17"/>
        <field name="tax_dest_id" ref="tax_template_in_icms_interno17"/>
    </record>

    <!-- Foreign sales -->
    <record id="account_fiscal_position_foreign_sale1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tax_template_out_icms_interno17"/>
        <field name="tax_dest_id" ref="tax_template_out_icms_externo"/>
    </record>

    <record id="account_fiscal_position_foreign_sale2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tax_template_out_icms_externo17"/>
        <field name="tax_dest_id" ref="tax_template_out_icms_externo"/>
    </record>

    <record id="account_fiscal_position_foreign_sale3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tax_template_out_ipi10"/>
        <field name="tax_dest_id" ref="tax_template_out_ipi"/>
    </record>

    <!-- Foreign purchases -->
    <record id="account_fiscal_position_foreign_purchase1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tax_template_in_icms_interno17"/>
        <field name="tax_dest_id" ref="tax_template_in_icms_interno"/>
    </record>

    <record id="account_fiscal_position_foreign_purchase2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tax_template_in_icms_externo17"/>
        <field name="tax_dest_id" ref="tax_template_in_icms_externo"/>
    </record>

    <record id="account_fiscal_position_foreign_purchase3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tax_template_in_ipi10"/>
        <field name="tax_dest_id" ref="tax_template_in_ipi"/>
    </record>

    <!-- Interstate sales between South/Southeast and North/Northeast/Midwest-->
    <record id="account_fiscal_position_interstate_sale1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ss_nnm"/>
        <field name="tax_src_id" ref="tax_template_out_icms_externo17"/>
        <field name="tax_dest_id" ref="tax_template_out_icms_externo7"/>
    </record>

    <record id="account_fiscal_position_interstate_sale2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ss_nnm"/>
        <field name="tax_src_id" ref="tax_template_out_icms_interno17"/>
        <field name="tax_dest_id" ref="tax_template_out_icms_externo7"/>
    </record>

    <!-- Interstate purchases between South/Southeast and North/Northeast/Midwest-->
    <record id="account_fiscal_position_interstate_purchase1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ss_nnm"/>
        <field name="tax_src_id" ref="tax_template_in_icms_externo17"/>
        <field name="tax_dest_id" ref="tax_template_in_icms_externo7"/>
    </record>

    <record id="account_fiscal_position_interstate_purchase2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ss_nnm"/>
        <field name="tax_src_id" ref="tax_template_in_icms_interno17"/>
        <field name="tax_dest_id" ref="tax_template_in_icms_externo7"/>
    </record>

    <!-- Interstate sales -->
    <record id="account_fiscal_position_interstate_sale3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_interstate"/>
        <field name="tax_src_id" ref="tax_template_out_icms_externo17"/>
        <field name="tax_dest_id" ref="tax_template_out_icms_externo12"/>
    </record>

    <record id="account_fiscal_position_interstate_sale4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_interstate"/>
        <field name="tax_src_id" ref="tax_template_out_icms_interno17"/>
        <field name="tax_dest_id" ref="tax_template_out_icms_externo12"/>
    </record>

    <!-- Interstate purchases -->
    <record id="account_fiscal_position_interstate_purchase3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_interstate"/>
        <field name="tax_src_id" ref="tax_template_in_icms_externo17"/>
        <field name="tax_dest_id" ref="tax_template_in_icms_externo12"/>
    </record>

    <record id="account_fiscal_position_interstate_purchase4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_interstate"/>
        <field name="tax_src_id" ref="tax_template_in_icms_interno17"/>
        <field name="tax_dest_id" ref="tax_template_in_icms_externo12"/>
    </record>

</odoo>

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
        <record id="tax_group_icms_17" model="account.tax.group">
            <field name="name">ICMS 17%</field>
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
        <record id="tax_group_issqn_2" model="account.tax.group">
            <field name="name">ISSQN 2%</field>
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
        <record id="tax_group_ipi_10" model="account.tax.group">
            <field name="name">IPI 10%</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ii_0" model="account.tax.group">
            <field name="name">II</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_inss_0" model="account.tax.group">
            <field name="name">INSS</field>
            <field name="country_id" ref="base.br"/>
        </record>

        <!-- New goods groups -->
        <record id="tax_group_aproxtrib_fed_incl_goods" model="account.tax.group">
            <field name="name">Tributação Federal Aproximada Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_aproxtrib_fed_excl_goods" model="account.tax.group">
            <field name="name">Tributação Federal Aproximada Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_aproxtrib_state_incl_goods" model="account.tax.group">
            <field name="name">Tributação Estadual Aproximada Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_aproxtrib_state_excl_goods" model="account.tax.group">
            <field name="name">Tributação Estadual Aproximada Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_cofins_incl_goods" model="account.tax.group">
            <field name="name">COFINS Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_cofins_excl_goods" model="account.tax.group">
            <field name="name">COFINS Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_cofins_deson_incl_goods" model="account.tax.group">
            <field name="name">COFINS Desoneração Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_cofins_deson_excl_goods" model="account.tax.group">
            <field name="name">COFINS Desoneração Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_cofins_st_incl_goods" model="account.tax.group">
            <field name="name">COFINS ST Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_cofins_st_excl_goods" model="account.tax.group">
            <field name="name">COFINS ST Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_incl_goods" model="account.tax.group">
            <field name="name">ICMS Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_excl_goods" model="account.tax.group">
            <field name="name">ICMS Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_credsn_incl_goods" model="account.tax.group">
            <field name="name">ICMS CredSN Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_credsn_excl_goods" model="account.tax.group">
            <field name="name">ICMS CredSN Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_deson_incl_goods" model="account.tax.group">
            <field name="name">ICMS Desoneração Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_deson_excl_goods" model="account.tax.group">
            <field name="name">ICMS Desoneração Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_difa_dest_incl_goods" model="account.tax.group">
            <field name="name">ICMS DIFA Destinatário Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_difa_dest_excl_goods" model="account.tax.group">
            <field name="name">ICMS DIFA Destinatário Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_difa_fcp_incl_goods" model="account.tax.group">
            <field name="name">ICMS DIFA FCP Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_difa_fcp_excl_goods" model="account.tax.group">
            <field name="name">ICMS DIFA FCP Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_difa_remet_incl_goods" model="account.tax.group">
            <field name="name">ICMS DIFA Remetente Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_difa_remet_excl_goods" model="account.tax.group">
            <field name="name">ICMS DIFA Remetente Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_eff_incl_goods" model="account.tax.group">
            <field name="name">ICMS EFF Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_eff_excl_goods" model="account.tax.group">
            <field name="name">ICMS EFF Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_fcp_incl_goods" model="account.tax.group">
            <field name="name">ICMS FCP Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_fcp_excl_goods" model="account.tax.group">
            <field name="name">ICMS FCP Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_own_payer_incl_goods" model="account.tax.group">
            <field name="name">ICMS Próprio Emitente Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_own_payer_excl_goods" model="account.tax.group">
            <field name="name">ICMS Próprio Emitente Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_part_incl_goods" model="account.tax.group">
            <field name="name">ICMS Partilha Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_part_excl_goods" model="account.tax.group">
            <field name="name">ICMS Partilha Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_rf_incl_goods" model="account.tax.group">
            <field name="name">ICMS RF Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_rf_excl_goods" model="account.tax.group">
            <field name="name">ICMS RF Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_incl_goods" model="account.tax.group">
            <field name="name">ICMS ST Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_excl_goods" model="account.tax.group">
            <field name="name">ICMS ST Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_fcp_incl_goods" model="account.tax.group">
            <field name="name">ICMS ST FCP Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_fcp_excl_goods" model="account.tax.group">
            <field name="name">ICMS ST FCP Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_fcppart_incl_goods" model="account.tax.group">
            <field name="name">ICMS ST FCP Partilha Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_fcppart_excl_goods" model="account.tax.group">
            <field name="name">ICMS ST FCP Partilha Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_part_incl_goods" model="account.tax.group">
            <field name="name">ICMS ST Partilha Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_part_excl_goods" model="account.tax.group">
            <field name="name">ICMS ST Partilha Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_sd_incl_goods" model="account.tax.group">
            <field name="name">ICMS ST SD Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_sd_excl_goods" model="account.tax.group">
            <field name="name">ICMS ST SD Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_sd_fcp_incl_goods" model="account.tax.group">
            <field name="name">ICMS ST SD FCP Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_icms_st_sd_fcp_excl_goods" model="account.tax.group">
            <field name="name">ICMS ST SD FCP Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ii_incl_goods" model="account.tax.group">
            <field name="name">II - Imposto de Importação Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ii_excl_goods" model="account.tax.group">
            <field name="name">II - Imposto de Importação Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_iof_incl_goods" model="account.tax.group">
            <field name="name">IOF - Imposto sobre Operações Financeiras Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_iof_excl_goods" model="account.tax.group">
            <field name="name">IOF - Imposto sobre Operações Financeiras Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_incl_goods" model="account.tax.group">
            <field name="name">IPI Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_excl_goods" model="account.tax.group">
            <field name="name">IPI Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_returned_incl_goods" model="account.tax.group">
            <field name="name">IPI Retornado Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_ipi_returned_excl_goods" model="account.tax.group">
            <field name="name">IPI Retornado Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_pis_incl_goods" model="account.tax.group">
            <field name="name">PIS Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_pis_excl_goods" model="account.tax.group">
            <field name="name">PIS Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_pis_deson_incl_goods" model="account.tax.group">
            <field name="name">PIS Desoneração Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_pis_deson_excl_goods" model="account.tax.group">
            <field name="name">PIS Desoneração Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_pis_st_incl_goods" model="account.tax.group">
            <field name="name">PIS ST Incl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
        <record id="tax_group_pis_st_excl_goods" model="account.tax.group">
            <field name="name">PIS ST Excl.</field>
            <field name="country_id" ref="base.br"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.br"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_icms" model="account.report.line">
                <field name="name">ICMS</field>
                <field name="aggregation_formula">TRIBUTADA_INTEGRALMENTE.balance + TRIBUTADA_E_COM_COBRANCA_DO_ICMS_POR_SUBSTITUICAO_TRIBUTARIA.balance</field>
                <field name="children_ids">
                    <record id="tax_report_icms_tributada" model="account.report.line">
                        <field name="name">Tributada integralmente</field>
                        <field name="code">TRIBUTADA_INTEGRALMENTE</field>
                        <field name="aggregation_formula">ICMS_1.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_icms_1" model="account.report.line">
                                <field name="name">ICMS base</field>
                                <field name="code">ICMS_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_icms_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ICMS_1</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_icms_tributada_com" model="account.report.line">
                        <field name="name">Tributada e com cobrança do ICMS por substituição tributária</field>
                        <field name="code">TRIBUTADA_E_COM_COBRANCA_DO_ICMS_POR_SUBSTITUICAO_TRIBUTARIA</field>
                        <field name="aggregation_formula">ICMS_2.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_icms_2" model="account.report.line">
                                <field name="name">ICMS tax</field>
                                <field name="code">ICMS_2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_icms_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ICMS_2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_icmsst" model="account.report.line">
                <field name="name">ICMS Subist</field>
                <field name="aggregation_formula">ICMSST_1.balance + ICMSST_2.balance</field>
                <field name="children_ids">
                    <record id="tax_report_icmsst_1" model="account.report.line">
                        <field name="name">ICMSST base</field>
                        <field name="code">ICMSST_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_icmsst_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ICMSST_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_icmsst_2" model="account.report.line">
                        <field name="name">ICMSST tax</field>
                        <field name="code">ICMSST_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_icmsst_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ICMSST_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_irpj" model="account.report.line">
                <field name="name">IRPJ</field>
                <field name="aggregation_formula">IRPJ_1.balance + IRPJ_2.balance</field>
                <field name="children_ids">
                    <record id="tax_report_irpj_1" model="account.report.line">
                        <field name="name">IRPJ base</field>
                        <field name="code">IRPJ_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_irpj_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">IRPJ_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_irpj_2" model="account.report.line">
                        <field name="name">IRPJ tax</field>
                        <field name="code">IRPJ_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_irpj_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">IRPJ_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ir" model="account.report.line">
                <field name="name">IR</field>
                <field name="aggregation_formula">IR_1.balance + IR_2.balance</field>
                <field name="children_ids">
                    <record id="tax_report_ir_1" model="account.report.line">
                        <field name="name">IR base</field>
                        <field name="code">IR_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">IR_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_2" model="account.report.line">
                        <field name="name">IR tax</field>
                        <field name="code">IR_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">IR_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_issqn" model="account.report.line">
                <field name="name">ISSQN</field>
                <field name="aggregation_formula">ISSQN_1.balance + ISSQN_2.balance</field>
                <field name="children_ids">
                    <record id="tax_report_issqn_1" model="account.report.line">
                        <field name="name">ISSQN base</field>
                        <field name="code">ISSQN_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_issqn_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ISSQN_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_issqn_2" model="account.report.line">
                        <field name="name">ISSQN tax</field>
                        <field name="code">ISSQN_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_issqn_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ISSQN_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_csll" model="account.report.line">
                <field name="name">CSLL</field>
                <field name="aggregation_formula">CSLL_1.balance + CSLL_2.balance</field>
                <field name="children_ids">
                    <record id="tax_report_csll_1" model="account.report.line">
                        <field name="name">CSLL base</field>
                        <field name="code">CSLL_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_csll_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CSLL_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_csll_2" model="account.report.line">
                        <field name="name">CSLL tax</field>
                        <field name="code">CSLL_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_csll_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CSLL_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_cofins" model="account.report.line">
                <field name="name">COFINS</field>
                <field name="aggregation_formula">COFINS_OPERACAO_TRIBUTAVEL_COM_ALIQUOTA_BASICA.balance</field>
                <field name="children_ids">
                    <record id="tax_report_cofins_oper_bas" model="account.report.line">
                        <field name="name">Operação Tributável com Alíquota Básica</field>
                        <field name="code">COFINS_OPERACAO_TRIBUTAVEL_COM_ALIQUOTA_BASICA</field>
                        <field name="aggregation_formula">COFINS_1.balance + COFINS_2.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_cofins_1" model="account.report.line">
                                <field name="name">COFINS base</field>
                                <field name="code">COFINS_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_cofins_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">COFINS_1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_cofins_2" model="account.report.line">
                                <field name="name">COFINS tax</field>
                                <field name="code">COFINS_2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_cofins_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">COFINS_2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_pis" model="account.report.line">
                <field name="name">PIS</field>
                <field name="aggregation_formula">PIS_OPERACAO_TRIBUTAVEL_COM_ALIQUOTA_BASICA.balance</field>
                <field name="children_ids">
                    <record id="tax_report_pis_oper_tri_basica" model="account.report.line">
                        <field name="name">Operação Tributável com Alíquota Básica</field>
                        <field name="code">PIS_OPERACAO_TRIBUTAVEL_COM_ALIQUOTA_BASICA</field>
                        <field name="aggregation_formula">PIS_1.balance + PIS_2.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_pis_1" model="account.report.line">
                                <field name="name">PIS base</field>
                                <field name="code">PIS_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_pis_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">PIS_1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_pis_2" model="account.report.line">
                                <field name="name">PIS tax</field>
                                <field name="code">PIS_2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_pis_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">PIS_2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ipi" model="account.report.line">
                <field name="name">IPI</field>
                <field name="code">BRTAX07</field>
                <field name="aggregation_formula">ENTRADA_COM_RECUPERACAO_DE_CREDITO.balance + ENTRADA_TRIBUTADA_COM_ALIQUOTA_ZERO.balance</field>
                <field name="children_ids">
                    <record id="tax_report_ipi_extrada_com" model="account.report.line">
                        <field name="name">Entrada com recuperação de crédito</field>
                        <field name="code">ENTRADA_COM_RECUPERACAO_DE_CREDITO</field>
                        <field name="aggregation_formula">BRTAX07_1.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_ipi_1" model="account.report.line">
                                <field name="name">IPI base</field>
                                <field name="code">BRTAX07_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ipi_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IPI_1</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ipi_extrada_tributada" model="account.report.line">
                        <field name="name">Entrada tributada com alíquota zero</field>
                        <field name="code">ENTRADA_TRIBUTADA_COM_ALIQUOTA_ZERO</field>
                        <field name="aggregation_formula">IPI_2.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_ipi_2" model="account.report.line">
                                <field name="name">IPI tax</field>
                                <field name="code">IPI_2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ipi_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IPI_2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ii" model="account.report.line">
                <field name="name">II</field>
                <field name="aggregation_formula">II_1.balance + II_2.balance</field>
                <field name="children_ids">
                    <record id="tax_report_ii_1" model="account.report.line">
                        <field name="name">II base</field>
                        <field name="code">II_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_ii_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ii_2" model="account.report.line">
                        <field name="name">II tax</field>
                        <field name="code">II_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_ii_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_inss" model="account.report.line">
                <field name="name">INSS</field>
                <field name="aggregation_formula">INSS_1.balance + INSS_2.balance</field>
                <field name="children_ids">
                    <record id="tax_report_inss_1" model="account.report.line">
                        <field name="name">INSS base</field>
                        <field name="code">INSS_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_inss_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">INSS_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_inss_2" model="account.report.line">
                        <field name="name">INSS tax</field>
                        <field name="code">INSS_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_inss_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">INSS_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_template_out_icms_interno17" model="account.tax.template">
        <field name="description">ICMS Interno 17%</field>
        <field name="name">ICMS Saída Interno 17%</field>
        <field name="amount">17</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020302'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_externo17" model="account.tax.template">
        <field name="description">ICMS Externo 17%</field>
        <field name="name">ICMS Saída Externo 17%</field>
        <field name="amount">17</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020302'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_in_icms_interno17" model="account.tax.template">
        <field name="description">ICMS Interno 17%</field>
        <field name="name">ICMS Entrada Interno 17%</field>
        <field name="amount">17</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020302'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_in_icms_externo17" model="account.tax.template">
        <field name="description">ICMS Externo 17%</field>
        <field name="name">ICMS Entrada Externo 17%</field>
        <field name="amount">17</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_17"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020302'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_interno" model="account.tax.template">
        <field name="description">ICMS Interno</field>
        <field name="name">ICMS Saída Interno 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_icms_externo" model="account.tax.template">
        <field name="description">ICMS Externo</field>
        <field name="name">ICMS Saída Externo 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_icms_interno" model="account.tax.template">
        <field name="description">ICMS Interno</field>
        <field name="name">ICMS Saída Interno 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_icms_externo" model="account.tax.template">
        <field name="description">ICMS Externo</field>
        <field name="name">ICMS Saída Externo 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_in_icms_interno" model="account.tax.template">
        <field name="description">ICMS Interno</field>
        <field name="name">ICMS Entrada Interno 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_in_icms_externo" model="account.tax.template">
        <field name="description">ICMS Externo</field>
        <field name="name">ICMS Entrada Externo 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_icms_externo7" model="account.tax.template">
        <field name="description">ICMS Externo 7%</field>
        <field name="name">ICMS Saída Externo 7%</field>
        <field name="amount">7</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020302'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_in_icms_externo7" model="account.tax.template">
        <field name="description">ICMS Externo 7%</field>
        <field name="name">ICMS Entrada Externo 7%</field>
        <field name="amount">7</field>
        <field name="type_tax_use">purchase</field>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020302'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_externo12" model="account.tax.template">
        <field name="description">ICMS Externo 12%</field>
        <field name="name">ICMS Saída Externo 12%</field>
        <field name="amount">12</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_12"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020302'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_in_icms_externo12" model="account.tax.template">
        <field name="description">ICMS Externo 12%</field>
        <field name="name">ICMS Entrada Externo 12%</field>
        <field name="amount">12</field>
        <field name="type_tax_use">purchase</field>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_12"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020302'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_subist" model="account.tax.template">
        <field name="description">ICMS Subist</field>
        <field name="name">ICMS Saída Subist 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="0" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icmsst_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icmsst_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_in_icms_subist" model="account.tax.template">
        <field name="description">ICMS Subist</field>
        <field name="name">ICMS Entrada Subist 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="0" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_icms_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icmsst_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icmsst_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
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
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'plus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020301'),
                'minus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
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
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020301'),
                'minus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'plus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_ipi" model="account.tax.template">
        <field name="description">IPI</field>
        <field name="name">IPI Saída 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="0" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_ipi_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_in_ipi" model="account.tax.template">
        <field name="description">IPI</field>
        <field name="name">IPI Entrada 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="0" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_ipi_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_pis" model="account.tax.template">
        <field name="description">PIS</field>
        <field name="name">PIS Saída 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_pis_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_pis065" model="account.tax.template">
        <field name="description">PIS 0,65%</field>
        <field name="name">PIS Saída 0,65%</field>
        <field name="amount">0.65</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_pis_065"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020303'),
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_in_pis" model="account.tax.template">
        <field name="description">PIS</field>
        <field name="name">PIS Entrada 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_pis_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
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
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020303'),
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_cofins" model="account.tax.template">
        <field name="description">COFINS</field>
        <field name="name">COFINS Saída 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_cofins_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_cofins3" model="account.tax.template">
        <field name="description">COFINS 3%</field>
        <field name="name">COFINS Saída 3%</field>
        <field name="amount">3</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_cofins_3"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'plus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020305'),
                'minus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_in_cofins" model="account.tax.template">
        <field name="description">COFINS</field>
        <field name="name">COFINS Entrada 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_cofins_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
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
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020305'),
                'minus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'plus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_irpj" model="account.tax.template">
        <field name="description">IRPJ</field>
        <field name="name">IRPJ 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_irpj_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_irpj_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_irpj_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_ir" model="account.tax.template">
        <field name="description">IR</field>
        <field name="name">IR 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_ir_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ir_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ir_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_issqn2" model="account.tax.template">
        <field name="description">ISSQN 2%</field>
        <field name="name">ISSQN Saída 2%</field>
        <field name="amount">2</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_issqn_2"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_issqn_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020340'),
                'plus_report_expression_ids': [ref('tax_report_issqn_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_issqn_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
                'minus_report_expression_ids': [ref('tax_report_issqn_2_tag')],
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
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_issqn_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_101020340'),
                'minus_report_expression_ids': [ref('tax_report_issqn_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_issqn_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
                'plus_report_expression_ids': [ref('tax_report_issqn_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_csll" model="account.tax.template">
        <field name="description">CSLL</field>
        <field name="name">CSLL 0%</field>
        <field name="amount">0.00</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="1" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_csll_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_csll_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_csll_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_ii0" model="account.tax.template">
        <field name="description">II</field>
        <field name="name">II Saída 0%</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="0" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_ii_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ii_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ii_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_in_ii0" model="account.tax.template">
        <field name="description">II</field>
        <field name="name">II Entrada 0%</field>
        <field name="amount">0</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="0" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_ii_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ii_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ii_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_out_inss0" model="account.tax.template">
        <field name="description">INSS</field>
        <field name="name">INSS Saída 0%</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field eval="0" name="price_include"/>
        <field eval="0" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_inss_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_inss_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_inss_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax_template_in_inss0" model="account.tax.template">
        <field name="description">INSS</field>
        <field name="name">INSS Entrada 0%</field>
        <field name="amount">0</field>
        <field name="type_tax_use">purchase</field>
        <field eval="0" name="price_include"/>
        <field eval="0" name="tax_discount"/>
        <field ref="l10n_br_account_chart_template" name="chart_template_id"/>
        <field name="tax_group_id" ref="tax_group_inss_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_inss_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_inss_1_tag')],
            }),
            (0, 0, {'repartition_type': 'tax'}),
        ]"/>
    </record>

    <!-- New goods taxes -->
    <record id="tax_template_out_aproxtrib_fed_incl_goods" model="account.tax.template">
        <field name="description">Tributação Federal Aproximada Incl.</field>
        <field name="name">Tributação Federal Aproximada Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_aproxtrib_fed_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
            }),
        ]"/>
    </record>
    <record id="tax_template_out_aproxtrib_fed_excl_goods" model="account.tax.template">
        <field name="description">Tributação Federal Aproximada Excl.</field>
        <field name="name">Tributação Federal Aproximada Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_aproxtrib_fed_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
            }),
        ]"/>
    </record>
    <record id="tax_template_out_aproxtrib_state_incl_goods" model="account.tax.template">
        <field name="description">Tributação Estadual Aproximada Incl.</field>
        <field name="name">Tributação Estadual Aproximada Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_aproxtrib_state_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
            }),
        ]"/>
    </record>
    <record id="tax_template_out_aproxtrib_state_excl_goods" model="account.tax.template">
        <field name="description">Tributação Estadual Aproximada Excl.</field>
        <field name="name">Tributação Estadual Aproximada Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_aproxtrib_state_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
            }),
        ]"/>
    </record>
    <record id="tax_template_out_cofins_incl_goods" model="account.tax.template">
        <field name="description">COFINS Incl.</field>
        <field name="name">COFINS Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_cofins_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'plus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'minus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_cofins_excl_goods" model="account.tax.template">
        <field name="description">COFINS Excl.</field>
        <field name="name">COFINS Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_cofins_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'plus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'minus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_cofins_deson_incl_goods" model="account.tax.template">
        <field name="description">COFINS Desoneração Incl.</field>
        <field name="name">COFINS Desoneração Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_cofins_deson_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'plus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'minus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_cofins_deson_excl_goods" model="account.tax.template">
        <field name="description">COFINS Desoneração Excl.</field>
        <field name="name">COFINS Desoneração Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_cofins_deson_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'plus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'minus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_cofins_st_incl_goods" model="account.tax.template">
        <field name="description">COFINS ST Incl.</field>
        <field name="name">COFINS ST Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_cofins_st_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'plus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'minus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_cofins_st_excl_goods" model="account.tax.template">
        <field name="description">COFINS ST Excl.</field>
        <field name="name">COFINS ST Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_cofins_st_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'plus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_cofins_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010905'),
                'minus_report_expression_ids': [ref('tax_report_cofins_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_incl_goods" model="account.tax.template">
        <field name="description">ICMS Incl.</field>
        <field name="name">ICMS Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_excl_goods" model="account.tax.template">
        <field name="description">ICMS Excl.</field>
        <field name="name">ICMS Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_credsn_incl_goods" model="account.tax.template">
        <field name="description">ICMS CredSN Incl.</field>
        <field name="name">ICMS CredSN Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_credsn_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_credsn_excl_goods" model="account.tax.template">
        <field name="description">ICMS CredSN Excl.</field>
        <field name="name">ICMS CredSN Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_credsn_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_deson_incl_goods" model="account.tax.template">
        <field name="description">ICMS Desoneração Incl.</field>
        <field name="name">ICMS Desoneração Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_deson_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_deson_excl_goods" model="account.tax.template">
        <field name="description">ICMS Desoneração Excl.</field>
        <field name="name">ICMS Desoneração Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_deson_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_difa_dest_incl_goods" model="account.tax.template">
        <field name="description">ICMS DIFA Destinatário Incl.</field>
        <field name="name">ICMS DIFA Destinatário Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_difa_dest_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_difa_dest_excl_goods" model="account.tax.template">
        <field name="description">ICMS DIFA Destinatário Excl.</field>
        <field name="name">ICMS DIFA Destinatário Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_difa_dest_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_difa_fcp_incl_goods" model="account.tax.template">
        <field name="description">ICMS DIFA FCP Incl.</field>
        <field name="name">ICMS DIFA FCP Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_difa_fcp_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_difa_fcp_excl_goods" model="account.tax.template">
        <field name="description">ICMS DIFA FCP Excl.</field>
        <field name="name">ICMS DIFA FCP Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_difa_fcp_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_difa_remet_incl_goods" model="account.tax.template">
        <field name="description">ICMS DIFA Remetente Incl.</field>
        <field name="name">ICMS DIFA Remetente Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_difa_remet_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_difa_remet_excl_goods" model="account.tax.template">
        <field name="description">ICMS DIFA Remetente Excl.</field>
        <field name="name">ICMS DIFA Remetente Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_difa_remet_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_eff_incl_goods" model="account.tax.template">
        <field name="description">ICMS EFF Incl.</field>
        <field name="name">ICMS EFF Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_eff_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_eff_excl_goods" model="account.tax.template">
        <field name="description">ICMS EFF Excl.</field>
        <field name="name">ICMS EFF Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_eff_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_fcp_incl_goods" model="account.tax.template">
        <field name="description">ICMS FCP Incl.</field>
        <field name="name">ICMS FCP Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_fcp_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_fcp_excl_goods" model="account.tax.template">
        <field name="description">ICMS FCP Excl.</field>
        <field name="name">ICMS FCP Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_fcp_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_own_payer_incl_goods" model="account.tax.template">
        <field name="description">ICMS Próprio Emitente Incl.</field>
        <field name="name">ICMS Próprio Emitente Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_own_payer_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_own_payer_excl_goods" model="account.tax.template">
        <field name="description">ICMS Próprio Emitente Excl.</field>
        <field name="name">ICMS Próprio Emitente Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_own_payer_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_part_incl_goods" model="account.tax.template">
        <field name="description">ICMS Partilha Incl.</field>
        <field name="name">ICMS Partilha Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_part_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_part_excl_goods" model="account.tax.template">
        <field name="description">ICMS Partilha Excl.</field>
        <field name="name">ICMS Partilha Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_part_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_rf_incl_goods" model="account.tax.template">
        <field name="description">ICMS RF Incl.</field>
        <field name="name">ICMS RF Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_rf_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_rf_excl_goods" model="account.tax.template">
        <field name="description">ICMS RF Excl.</field>
        <field name="name">ICMS RF Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_rf_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_incl_goods" model="account.tax.template">
        <field name="description">ICMS ST Incl.</field>
        <field name="name">ICMS ST Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_excl_goods" model="account.tax.template">
        <field name="description">ICMS ST Excl.</field>
        <field name="name">ICMS ST Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_fcp_incl_goods" model="account.tax.template">
        <field name="description">ICMS ST FCP Incl.</field>
        <field name="name">ICMS ST FCP Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_fcp_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_fcp_excl_goods" model="account.tax.template">
        <field name="description">ICMS ST FCP Excl.</field>
        <field name="name">ICMS ST FCP Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_fcp_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_fcppart_incl_goods" model="account.tax.template">
        <field name="description">ICMS ST FCP Partilha Incl.</field>
        <field name="name">ICMS ST FCP Partilha Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_fcppart_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_fcppart_excl_goods" model="account.tax.template">
        <field name="description">ICMS ST FCP Partilha Excl.</field>
        <field name="name">ICMS ST FCP Partilha Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_fcppart_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_part_incl_goods" model="account.tax.template">
        <field name="description">ICMS ST Partilha Incl.</field>
        <field name="name">ICMS ST Partilha Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_part_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_part_excl_goods" model="account.tax.template">
        <field name="description">ICMS ST Partilha Excl.</field>
        <field name="name">ICMS ST Partilha Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_part_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_sd_incl_goods" model="account.tax.template">
        <field name="description">ICMS ST SD Incl.</field>
        <field name="name">ICMS ST SD Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_sd_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_sd_excl_goods" model="account.tax.template">
        <field name="description">ICMS ST SD Excl.</field>
        <field name="name">ICMS ST SD Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_sd_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_sd_fcp_incl_goods" model="account.tax.template">
        <field name="description">ICMS ST SD FCP Incl.</field>
        <field name="name">ICMS ST SD FCP Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_sd_fcp_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_icms_st_sd_fcp_excl_goods" model="account.tax.template">
        <field name="description">ICMS ST SD FCP Excl.</field>
        <field name="name">ICMS ST SD FCP Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_icms_st_sd_fcp_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'plus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_icms_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010903'),
                'minus_report_expression_ids': [ref('tax_report_icms_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_ii_incl_goods" model="account.tax.template">
        <field name="description">II - Imposto de Importação Incl.</field>
        <field name="name">II - Imposto de Importação Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_ii_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ii_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
                'plus_report_expression_ids': [ref('tax_report_ii_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ii_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
                'minus_report_expression_ids': [ref('tax_report_ii_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_ii_excl_goods" model="account.tax.template">
        <field name="description">II - Imposto de Importação Excl.</field>
        <field name="name">II - Imposto de Importação Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_ii_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ii_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
                'plus_report_expression_ids': [ref('tax_report_ii_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ii_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010928'),
                'minus_report_expression_ids': [ref('tax_report_ii_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_iof_incl_goods" model="account.tax.template">
        <field name="description">IOF - Imposto sobre Operações Financeiras Incl.</field>
        <field name="name">IOF - Imposto sobre Operações Financeiras Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_iof_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010906'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010906'),
            }),
        ]"/>
    </record>
    <record id="tax_template_out_iof_excl_goods" model="account.tax.template">
        <field name="description">IOF - Imposto sobre Operações Financeiras Excl.</field>
        <field name="name">IOF - Imposto sobre Operações Financeiras Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_iof_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010906'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010906'),
            }),
        ]"/>
    </record>
    <record id="tax_template_out_ipi_incl_goods" model="account.tax.template">
        <field name="description">IPI Incl.</field>
        <field name="name">IPI Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_ipi_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'plus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'minus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_ipi_excl_goods" model="account.tax.template">
        <field name="description">IPI Excl.</field>
        <field name="name">IPI Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_ipi_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'plus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'minus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_ipi_returned_incl_goods" model="account.tax.template">
        <field name="description">IPI Retornado Incl.</field>
        <field name="name">IPI Retornado Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_ipi_returned_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'minus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_ipi_returned_excl_goods" model="account.tax.template">
        <field name="description">IPI Retornado Excl.</field>
        <field name="name">IPI Retornado Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_ipi_returned_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_ipi_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010902'),
                'minus_report_expression_ids': [ref('tax_report_ipi_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_pis_incl_goods" model="account.tax.template">
        <field name="description">PIS Incl.</field>
        <field name="name">PIS Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_pis_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_pis_excl_goods" model="account.tax.template">
        <field name="description">PIS Excl.</field>
        <field name="name">PIS Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_pis_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_pis_deson_incl_goods" model="account.tax.template">
        <field name="description">PIS Desoneração Incl.</field>
        <field name="name">PIS Desoneração Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_pis_deson_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_pis_deson_excl_goods" model="account.tax.template">
        <field name="description">PIS Desoneração Excl.</field>
        <field name="name">PIS Desoneração Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_pis_deson_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_pis_st_incl_goods" model="account.tax.template">
        <field name="description">PIS ST Incl.</field>
        <field name="name">PIS ST Incl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="True"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_pis_st_incl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
    </record>
    <record id="tax_template_out_pis_st_excl_goods" model="account.tax.template">
        <field name="description">PIS ST Excl.</field>
        <field name="name">PIS ST Excl.</field>
        <field name="amount">1.00</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="price_include" eval="False"/>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="tax_group_id" ref="tax_group_pis_st_excl_goods"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'plus_report_expression_ids': [ref('tax_report_pis_2_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_pis_1_tag')],
            }),
            (0, 0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_201010904'),
                'minus_report_expression_ids': [ref('tax_report_pis_2_tag')],
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

## File: data\fiscal_templates_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Fiscal Position Templates -->
    <record id="fiscal_position_template_1" model="account.fiscal.position.template">
        <field name="sequence">1</field>
        <field name="name">Internal (within one state)</field>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_id" ref="base.br"/>
        <field name="l10n_br_fp_type">internal</field>
    </record>

    <record id="fiscal_position_template_2" model="account.fiscal.position.template">
        <field name="sequence">2</field>
        <field name="name">Foreign</field>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="auto_apply" eval="True"/>
    </record>

    <!-- South and Southeast company delivering to North, Northeast, and Midwest -->
    <record id="fiscal_position_template_ss_nnm" model="account.fiscal.position.template">
        <field name="sequence">3</field>
        <field name="name">South and Southeast to North, Northeast, and Midwest</field>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_id" ref="base.br"/>
        <field name="l10n_br_fp_type">ss_nnm</field>
    </record>

    <!-- Other interstate transactions -->
    <record id="fiscal_position_template_interstate" model="account.fiscal.position.template">
        <field name="sequence">4</field>
        <field name="name">Interstate</field>
        <field name="chart_template_id" ref="l10n_br_account_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_id" ref="base.br"/>
        <field name="l10n_br_fp_type">interstate</field>
    </record>

    <!-- Fiscal Position Account Templates -->
    <record id="fiscal_position_account_template_2" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_2" />
        <field name="account_src_id" ref="l10n_br.account_template_30101010105" />
        <field name="account_dest_id" ref="l10n_br.account_template_30101010101" />
    </record>

    <record id="fiscal_position_account_template_3" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_2" />
        <field name="account_src_id" ref="l10n_br.account_template_30101010106" />
        <field name="account_dest_id" ref="l10n_br.account_template_30101010103" />
    </record>
</odoo>

```

## File: data\l10n_br_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
	<record id="l10n_br_account_chart_template" model="account.chart.template">
		<field name="name">Plano de Contas Brasileiro</field>
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

## File: models\account_chart_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _get_fp_vals(self, company, position):
        res = super()._get_fp_vals(company, position)
        if company.country_id.code == 'BR':
            res['l10n_br_fp_type'] = position['l10n_br_fp_type']
        return res

```

## File: models\account_fiscal_position.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models

SOUTH_SOUTHEAST = {"PR", "RS", "SC", "SP", "ES", "MG", "RJ"}
NORTH_NORTHEAST_MIDWEST = {
    "AC", "AP", "AM", "PA", "RO", "RR", "TO", "AL", "BA", "CE",
    "MA", "PB", "PE", "PI", "RN", "SE", "DF", "GO", "MT", "MS"
}


class AccountFiscalPosition(models.Model):
    _inherit = 'account.fiscal.position'

    l10n_br_fp_type = fields.Selection(
        selection=[
            ('internal', 'Internal'),
            ('ss_nnm', 'South/Southeast selling to North/Northeast/Midwest'),
            ('interstate', 'Other interstate'),
        ],
        string='Interstate Fiscal Position Type',
    )

    @api.model
    def _get_fiscal_position(self, partner, delivery=None):
        if not delivery:
            delivery = partner

        if self.env.company.country_id.code != "BR" or delivery.country_id.code != 'BR':
            return super()._get_fiscal_position(partner, delivery=delivery)

        # manually set fiscal position on partner has a higher priority
        manual_fiscal_position = delivery.property_account_position_id or partner.property_account_position_id
        if manual_fiscal_position:
            return manual_fiscal_position

        # Taxation in Brazil depends on both the state of the partner and the state of the company
        if self.env.company.state_id == delivery.state_id:
            return self.search([('l10n_br_fp_type', '=', 'internal'), ('company_id', '=', self.env.company.id)], limit=1)
        if self.env.company.state_id.code in SOUTH_SOUTHEAST and delivery.state_id.code in NORTH_NORTHEAST_MIDWEST:
            return self.search([('l10n_br_fp_type', '=', 'ss_nnm'), ('company_id', '=', self.env.company.id)], limit=1)
        return self.search([('l10n_br_fp_type', '=', 'interstate'), ('company_id', '=', self.env.company.id)], limit=1)

```

## File: models\account_fiscal_position_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountFiscalPositionTemplate(models.Model):
    _inherit = 'account.fiscal.position.template'

    l10n_br_fp_type = fields.Selection(
        selection=[
            ('internal', 'Internal'),
            ('ss_nnm', 'South/Southeast selling to North/Northeast/Midwest'),
            ('interstate', 'Other interstate'),
        ],
        string='Interstate Fiscal Position Type',
    )

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = "res.company"

    # ==== Business fields ====
    l10n_br_cpf_code = fields.Char(string="CPF", help="Natural Persons Register.")
    l10n_br_ie_code = fields.Char(string="IE", help="State Tax Identification Number. Should contain 9-14 digits.") # each state has its own format. Not all of the validation rules can be easily found.
    l10n_br_im_code = fields.Char(string="IM", help="Municipal Tax Identification Number") # each municipality has its own format. There is no information about validation anywhere.
    l10n_br_nire_code = fields.Char(string="NIRE", help="State Commercial Identification Number. Should contain 11 digits.")

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
import re


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_br_cpf_code = fields.Char(string="CPF", help="Natural Persons Register.")
    l10n_br_ie_code = fields.Char(string="IE", help="State Tax Identification Number. Should contain 9-14 digits.")
    l10n_br_im_code = fields.Char(string="IM", help="Municipal Tax Identification Number")
    l10n_br_isuf_code = fields.Char(string="SUFRAMA code", help="SUFRAMA registration number.")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2009  Renato Lima - Akretion

from . import account
from . import account_fiscal_position_template
from . import account_fiscal_position
from . import account_chart_template
from . import res_company
from . import res_partner

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

## File: views\account_fiscal_position_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_position_form" model="ir.ui.view">
        <field name="name">account.fiscal.position.form</field>
        <field name="model">account.fiscal.position</field>
        <field name="inherit_id" ref="account.view_account_position_form"/>
        <field name="arch" type="xml">
            <field name="auto_apply" position="after">
                <field name="l10n_br_fp_type" options="{'no_open': True, 'no_create': True}" attrs="{'invisible': [('country_id', '!=', %(base.br)d)]}"/>
            </field>
        </field>
    </record>

</odoo>

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

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_company_form" model="ir.ui.view">
        <field name="name">res.company.form</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_br_cpf_code" attrs="{'invisible': [('country_id', '!=', %(base.br)d)]}"/>
                <field name="l10n_br_ie_code" attrs="{'invisible': [('country_id', '!=', %(base.br)d)]}"/>
                <field name="l10n_br_im_code" attrs="{'invisible': [('country_id', '!=', %(base.br)d)]}"/>
                <field name="l10n_br_nire_code" attrs="{'invisible': [('country_id', '!=', %(base.br)d)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="br_partner_tax_fields_form" model="ir.ui.view">
            <field name="name">res.partner.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="after">
                    <field name="l10n_br_cpf_code" attrs="{'invisible': [('country_id', '!=', %(base.br)d)]}"/>
                    <field name="l10n_br_ie_code" attrs="{'invisible': [('country_id', '!=', %(base.br)d)]}"/>
                    <field name="l10n_br_im_code" attrs="{'invisible': [('country_id', '!=', %(base.br)d)]}"/>
                    <field name="l10n_br_isuf_code" attrs="{'invisible': [('country_id', '!=', %(base.br)d)]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

