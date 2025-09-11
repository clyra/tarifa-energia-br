# tarifa-energia-br
Scripts python para buscar o valor da tarifa de energia da base de dados da aneel. Para a versão no formato integração vá para: https://github.com/vodikus/ha_tarifas_energia_brasil[integração ha_tarifas_energia_brasil].

## Introdução

A ANEEL disponibiliza através da plataforma dadosabertos.aneel.gov.br vários dados, entre eles os de tarifa de energia elétrica e bandeiras tarifárias. Os scripts desse repositório conseguem extrair esses dados de maneira ainda bastante simples (e fragil...), mas a idéia é que possam ser melhorados/adaptados com o uso.

### tarifa.py

Esse script baixa e calcula o preço por KWh da modalidade residencial convencional de acordo com a operadora de energia. Consulte a base de dados da Aneel em https://dadosabertos.aneel.gov.br/dataset/tarifas-distribuidoras-energia-eletrica/resource/fcf2906c-7c32-4b9b-a637-054e7a5234f4 [tarifas-distribuidoras-energia] para consultar o nome da sua distribuidora (use a busca e veja o valor da coluna SigAgente). Modifique a variavel "operadora" do script com a sua operadora.

Caso vc use a modalidade de tarifa branca, ou de propriedade rural, etc, vai precisar modificar o script. Observe que o preço da tarifa é a soma da Tarifa de Energia + o Tarifa de Uso do Sistema de Distribuição (o script já te dá o valor somado, mas pode ser modificado para mostrar os valores de maneira separada).


## bandeira.py

Esse script consulta a bandeira vigente e o valor adicional do KWh. Ele devolve um json com esses dois atributos. 

## Como usar no Homeassistant

Um maneira de usar esses scripts no homeassistant é usa-los como sensores do tipo command_line. Crie um folder "shell_scripts" no seu HA, coloque os scripts lá com permissão de execução e configure conforme o yaml abaixo:

`````
command_line:
  - sensor:
      name: Preço KWh 
      command: "/config/shell_scripts/tarifa.py"
      unit_of_measurement: "R$"
      scan_interval: 86400
  - sensor:
      name: Bandeira Tarifaria
      json_attributes:
        - valor 
      command: "/config/shell_scripts/bandeira.py"
      value_template: "{{ value_json.bandeira }}"
      scan_interval: 86400

automation:
  - alias: "Atualizar bandeira e tarifa"
    trigger:
      - platform: time
        at: "08:00:00"
    action:
      - service: homeassistant.update_entity
        target:
          entity_id: 
            - sensor.bandeira_tarifaria
            - sensor.preco_kwh

`````

Como essas valores mudam com pouca frequencia, recomendo que ou seja usando um "scan_interval" alto, ou que se desabilite o scan automatico e se crie uma automação para atualizar o valor uma vez por dia, como no exemplo acima. 
