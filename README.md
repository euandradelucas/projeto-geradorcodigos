# projeto-geradorcodigos
from nicegui import ui
import os
import random # implementar os códigos aleatórios para o produto.

BASE_DIR = os.path.dirname(os.path.abspath(__file__))

#Define os caminhos dos arquivos txt
cidades = os.path.join(BASE_DIR, "cidades.txt")
empresas = os.path.join(BASE_DIR, "empresas.txt")
interno = os.path.join(BASE_DIR, "interno.txt")
digito = os.path.join(BASE_DIR, "digito.txt")
ARQUIVO = os.path.join(BASE_DIR, "codigos.txt")

# Atribui um nome as cidades, para facilitar a visualização
cidades_nomes = {
    'FI': 'FOZ DO IGUAÇU',
    'CJ': 'CAMPOS DO JORDÃO',
    'GDO': 'GRAMADO',
    'BC': 'BALNEARIO CAMBORIÚ',
    'BLU': 'BLUMENAU',
    'OL': 'OLíMPIA',
    'PG': 'PORTO DE GALINHAS',
    'RJ': 'RIO DE JANEIRO',
}

# Atribui um nome aos códigos, para facilitar a visualização
digitos_nomes = {
    'B' : 'BALCÃO',
    'W' : 'WEB',
    'P': 'PRESENCIAL'
}

#Verfica se os arquivos existem, se não existir, cria um arquivo vazio
if not os.path.exists(ARQUIVO):
    with open(ARQUIVO, 'w', encoding='utf-8') as f:
        pass


def ler_opcoes(arquivo):
    try:
        with open(arquivo, 'r', encoding='utf-8') as f:
            opcoes = [linha.strip() for linha in f if linha.strip()]
        return opcoes
    except Exception as e:
        ui.label(f'Erro ao ler {arquivo}: {e}').style('color: red')
        ui.run()
        exit()

cidades_opcoes = ler_opcoes(cidades)
empresas_opcoes = ler_opcoes(empresas)
interno_opcoes = ler_opcoes(interno)
digito_opcoes = ler_opcoes(digito)

if not cidades_opcoes or not empresas_opcoes or not interno_opcoes or not digito_opcoes:
    ui.label('Algum arquivo está vazio!').style('color: red')
else:
    cidades_display = [cidades_nomes.get(c, c) for c in cidades_opcoes]

    cidade = ui.select(cidades_display, label='Cidade').style('width: 350px; margin-bottom: 8px;')
    empresa = ui.select(empresas_opcoes, label='Empresa').style('width: 350px; margin-bottom: 8px;')
    produto = ui.select(interno_opcoes, label='Produto').style('width: 350px; margin-bottom: 16px;')
    digito = ui.select(digito_opcoes, label='Dígito').style('width: 350px; margin-bottom: 16px;')
    resultado = ui.label('').style('font-size: 1.5rem; color: #1976d2; margin-top: 24px;')

    # Função usada para chamar o código da cidade!
    def get_codigo_cidade(nome):
        for codigo, nome_completo in cidades_nomes.items():
            if nome_completo == nome:
                return codigo
        return nome

    def criar():
        if not cidade.value or not empresa.value or not produto.value or not digito.value:
            ui.notify('Selecione todas as opções!', color='warning')
            return
        try:
            codigo_cidade = get_codigo_cidade(cidade.value)
            codigo_completo = f'{codigo_cidade}{empresa.value}{produto.value}{digito.value}'
            with open(ARQUIVO, 'a', encoding='utf-8') as f:
                f.write(f'{codigo_cidade},{empresa.value},{produto.value},{digito.value},{codigo_completo}\n')
            ui.notify(f'Registro criado! Código: {codigo_completo}', color='positive')
            buscar_codigos()  # Atualiza a tabela automaticamente
        except Exception as e:
            ui.notify(f'Erro ao salvar: {e}', color='negative')

    ui.button('Criar', on_click=criar).props('color=primary')

    tabela = ui.table(
        columns=[
            {'name': 'codigo', 'label': 'Código Completo', 'field': 'codigo', 'align': 'left'},
            {'name': 'cidade', 'label': 'Cidade', 'field': 'cidade', 'align': 'left'},
            {'name': 'empresa', 'label': 'Empresa', 'field': 'empresa', 'align': 'left'},
            {'name': 'produto', 'label': 'Produto', 'field': 'produto', 'align': 'left'},
            {'name': 'digito', 'label': 'Dígito', 'field': 'digito', 'align': 'left'},
        ],
        rows=[],
        row_key='id',
    ).style(
        'font-size: 1.2rem; min-width: 1300px; margin-top: 10px;'
        'padding: 8px 24px;'
    ).props('dense square flat bordered wrap-cells')

    def buscar_codigos():
        linhas = []
        if os.path.exists(ARQUIVO):
            with open(ARQUIVO, 'r', encoding='utf-8') as f:
                for idx, linha in enumerate(f):
                    partes = linha.strip().split(',')
                    if len(partes) >= 5:
                        cidade_nome = cidades_nomes.get(partes[0], partes[0])
                        linhas.append({
                            'id': idx,
                            'codigo': partes[4],
                            'cidade': cidade_nome,
                            'empresa': partes[1],
                            'produto': partes[2],
                            'digito': partes[3],
                        })
        tabela.rows = linhas
        tabela.update()

    ui.button('Buscar códigos criados', on_click=buscar_codigos).props('color=secondary')

    buscar_codigos() # Busca os códigos já ao iniciar a pagina

ui.run()
