from nicegui import ui
import os
import random

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


def read(arquivo):
    try:
        with open(arquivo, 'r', encoding='utf-8') as f:
            opt = [linha.strip() for linha in f if linha.strip()]
        return opt
    except Exception as e:
        ui.label(f'Erro ao ler {arquivo}: {e}').style('color: red')
        ui.run()
        exit()

cidades_opt = read(cidades)
empresas_opt = read(empresas)
interno_opt = read(interno) + ['Gerar Aleatório']  # Adiciona a opção de gerar aleatório
digitos_opt = read(digito)

if not all([cidades_opt, empresas_opt, interno_opt, digitos_opt]):
    ui.label('Algum arquivo .txt está vazio!').style('color: red')
else:
    # Busca os digitos das cidades numa lista e transforma os em nomes na tabela
    cidades_display = [cidades_nomes.get(c, c) for c in cidades_opt]

    cidade = ui.select(cidades_display, label='Cidade').style('width: 350px; margin-bottom: 8px;')
    empresa = ui.select(empresas_opt, label='Empresa').style('width: 350px; margin-bottom: 8px;')
    produto = ui.select(interno_opt, label='Produto').style('width: 350px; margin-bottom: 16px;')
    digito = ui.select(digitos_opt, label='Dígito').style('width: 350px; margin-bottom: 16px;')

    # Função usada para chamar o código da cidade!
    def get_codigo_cidade(nome):
        for codigo, nome_completo in cidades_nomes.items():
            if nome_completo == nome:
                return codigo
        return nome

    def criar():
        if not all([cidade.value, empresa.value, produto.value, digito.value]):
            ui.notify('Selecione todas as opções!', color='warning')
            return
        try:
            codigo_cidade = get_codigo_cidade(cidade.value)

            if produto.value == 'Gerar Aleatório':
                produto.valor = f"A{random.randint(1, 100):03}"
            else :
                produto.valor = produto.value

            codigo_completo = f'{codigo_cidade}{empresa.value}{produto.valor}{digito.value}'

            #Verifica se o código já exisete
            with open(ARQUIVO, 'a', encoding='utf-8') as f:
                f.write(f'{codigo_cidade},{empresa.value},{produto.valor},{digito.value},{codigo_completo}\n')
            ui.notify(f'Registro criado! Código: {codigo_completo}', color='positive')
            exibir_codigos()  # Atualiza a tabela automaticamente
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
    ).style(
        'font-size: 1.2rem; min-width: 1300px; margin-top: 10px;'
        'padding: 8px 24px;'
    ).props('dense square flat bordered wrap-cells')


# Função para buscar os códigos já criados e exibi-los na tabela
    def exibir_codigos():
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

    exibir_codigos() # Busca os códigos já ao iniciar a pagina

ui.run()
