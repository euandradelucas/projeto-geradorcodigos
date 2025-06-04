from nicegui import ui
import os
import random

BASE_DIR = os.path.dirname(os.path.abspath(__file__))

# Define os caminhos dos arquivos txt
cidades = os.path.join(BASE_DIR, "cidades.txt")
empresas = os.path.join(BASE_DIR, "empresas.txt")
interno = os.path.join(BASE_DIR, "interno.txt")
digito = os.path.join(BASE_DIR, "digito.txt")
ARQUIVO = os.path.join(BASE_DIR, "codigos.txt")

# Converte os nomes das cidades para facilitar a exibição
cidades_nomes = {
    'FI': 'FOZ DO IGUAÇU',
    'CJ': 'CAMPOS DO JORDÃO',
    'GDO': 'GRAMADO',
    'BC': 'BALNEARIO CAMBORIÚ',
    'BLU': 'BLUMENAU',
    'OL': 'OLÍMPIA',
    'PG': 'PORTO DE GALINHAS',
    'RJ': 'RIO DE JANEIRO',
}

# Converte os digitos para facilitar a exibição
digitos_nomes = {
    'B': 'BALCÃO',
    'W': 'WEB',
    'N': 'NET',
    'A': 'ANTECIPADO',
}

# Garante que o arquivo de códigos existe
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

# Lê os arquivos txt e armazena as opções
cidades_opt = read(cidades)
interno_opt = read(interno) + ['Gerar Aleatório'] # Gera um input a mais (GERAR ALEATÓRIO)
digitos_opt = read(digito)

if not all([cidades_opt, interno_opt, digitos_opt]):
    ui.label('Algum arquivo .txt está vazio!').style('color: red')
else:
    cidades_display = [cidades_nomes.get(c, c) for c in cidades_opt]
    digitos_display = [digitos_nomes.get(d, d) for d in digitos_opt]

    ui.label('🔑 Cadastrar Código').style(
        '''
        color: #4B2991;
        font-size: 2rem;
        font-weight: bold;
        letter-spacing: 1px;
        margin-bottom: 18px;
        margin-top: 28px;
        font-family: "Segoe UI", "Arial", sans-serif;
        '''
    )

    #Criação dos campos de entrada do usuário
    cidade = ui.select(cidades_display, label='Cidade').style('width: 350px; margin-bottom: 8px;')
    empresa = ui.input(label='Empresa').style('width: 350px; margin-bottom: 8px;')
    produto = ui.select(interno_opt, label='Produto').style('width: 350px; margin-bottom: 16px;')
    digito_sel = ui.select(digitos_display, label='Categoria').style('width: 350px; margin-bottom: 16px;')

    def get_codigo_cidade(nome):
        for codigo, nome_completo in cidades_nomes.items():
            if nome_completo == nome:
                return codigo
        return nome

    def get_codigo_digito(nome):
        for codigo, nome_completo in digitos_nomes.items():
            if nome_completo == nome:
                return codigo
        return nome

    # Método para criar o código
    def criar():
        if not all([cidade.value, empresa.value, produto.value, digito_sel.value]):
            ui.notify('Selecione todas as opções!', color='warning')
            return
        try:
            codigo_cidade = get_codigo_cidade(cidade.value)
            codigo_digito = get_codigo_digito(digito_sel.value)
            if produto.value == 'Gerar Aleatório':
                produto_valor = f"A{random.randint(1, 100):03}"
            else:
                produto_valor = produto.value
            codigo_completo = f'{codigo_cidade}{empresa.value}{produto_valor}{codigo_digito}'
            if os.path.exists(ARQUIVO):
                with open(ARQUIVO, 'r', encoding='utf-8') as f:
                    for linha in f:
                        partes = linha.strip().split(',')
                        if len(partes) >= 5 and partes[4] == codigo_completo:
                            ui.notify('Código já existe! Escolha outra combinação.', color='negative')
                            return
            with open(ARQUIVO, 'a', encoding='utf-8') as f:
                f.write(f'{codigo_cidade},{empresa.value},{produto_valor},{codigo_digito},{codigo_completo}\n')
            ui.notify(f'Registro criado! Código: {codigo_completo}', color='positive')
            exibir_codigos()
        except Exception as e:
            ui.notify(f'Erro ao salvar: {e}', color='negative')

    ui.button('Criar', on_click=criar).props('color=accent')

    #Tabela de códigos gerados
    ui.label('📦 Códigos Gerados').style(
        '''
        color: #4B2991;
        font-size: 1.5rem;
        font-weight: bold;
        letter-spacing: 1px;
        margin-bottom: 18px;
        margin-top: 28px;
        font-family: "Segoe UI", "Arial", sans-serif;
        '''
    )

    #Usada para exibir os códigos gerados
    tabela = ui.table(
        columns=[
            {'name': 'codigo', 'label': 'Código Completo', 'field': 'codigo', 'align': 'left'},
            {'name': 'cidade', 'label': 'Cidade', 'field': 'cidade', 'align': 'left'},
            {'name': 'empresa', 'label': 'Empresa', 'field': 'empresa', 'align': 'left'},
            {'name': 'produto', 'label': 'Produto', 'field': 'produto', 'align': 'left'},
            {'name': 'digito', 'label': 'Categoria', 'field': 'digito', 'align': 'left'},
        ],
        rows=[],
        row_key='id',
        selection='single',  # Checkbox para selecionar apenas uma linha
    ).style(
        'font-size: 1.2rem; min-width: 1400px; margin-top: 10px;'
        'padding: 8px 24px;'
    ).props('dense square flat bordered wrap-cells')


    # Método para copiar o código selecionado para a área de transferência
    def copiar_selecionados():
        codigos = [row['codigo'] for row in tabela.selected]
        if codigos:
            ui.run_javascript(
                f'navigator.clipboard.writeText("{chr(10).join(codigos)}");'
            )
            ui.notify('Código(s) copiado(s)!', color='positive')
        else:
            ui.notify('Selecione pelo menos um código!', color='warning')

    ui.button('Copiar Código Selecionado', on_click=copiar_selecionados).props('color=accent').style('margin-bottom: 16px;')

    # Método para exibir os códigos gerados na tabela
    def exibir_codigos():
        linhas = []
        if os.path.exists(ARQUIVO):
            with open(ARQUIVO, 'r', encoding='utf-8') as f:
                for idx, linha in enumerate(f):
                    partes = linha.strip().split(',')
                    if len(partes) >= 5:
                        cidade_nome = cidades_nomes.get(partes[0], partes[0])
                        digito_nome = digitos_nomes.get(partes[3], partes[3])
                        linhas.append({
                            'id': idx,
                            'codigo': partes[4],
                            'cidade': cidade_nome,
                            'empresa': partes[1],
                            'produto': partes[2],
                            'digito': digito_nome,
                        })
        tabela.rows = linhas
        tabela.update() # Atualiza a tabela com os dados lidos
    exibir_codigos() # Exibe os códigos ao iniciar

ui.run()
