from nicegui import ui
from database import Session, Codigo
import random

# Dicionários para exibição
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

digitos_nomes = {
    'B': 'BALCÃO',
    'W': 'WEB',
    'N': 'NET',
    'A': 'ANTECIPADO',
}

def main():
    session = Session()

    ui.label('🔑 Gerador de Códigos').style(
        '''
        color: #4B2991;
        font-size: 2rem;
        font-weight: bold;
        margin-bottom: 18px;
        margin-top: 28px;
        '''
    )

    cidade = ui.select(list(cidades_nomes.values()), label='Cidade')
    empresa = ui.input(label='Empresa')
    produto = ui.select(['A001', 'A002', 'Gerar Aleatório'], label='Produto')
    categoria = ui.select(list(digitos_nomes.values()), label='Categoria')

    for campo in [cidade, empresa, produto, categoria]:
        campo.style('width: 350px; margin-bottom: 8px;')

    tabela = ui.table(
        columns=[
            {'name': 'codigo', 'label': 'Código', 'field': 'codigo'},
            {'name': 'cidade', 'label': 'Cidade', 'field': 'cidade'},
            {'name': 'empresa', 'label': 'Empresa', 'field': 'empresa'},
            {'name': 'produto', 'label': 'Produto', 'field': 'produto'},
            {'name': 'categoria', 'label': 'Categoria', 'field': 'categoria'},
            {'name': 'data_criacao', 'label': 'Data Criação', 'field': 'data_criacao'},
        ],
        rows=[],
        row_key='id',
        selection='single',
    ).style('margin-top: 20px')

    def atualizar_tabela():
        codigos = session.query(Codigo).order_by(Codigo.created_at.desc()).all()
        tabela.rows = [c.to_dict() for c in codigos]
        tabela.update()

    def get_codigo_cidade(nome):
        for codigo, nome_completo in cidades_nomes.items():
            if nome_completo == nome:
                return codigo
        return nome

    def get_codigo_categoria(nome):
        for codigo, nome_completo in digitos_nomes.items():
            if nome_completo == nome:
                return codigo
        return nome

    def criar():
        if not all([cidade.value, empresa.value, produto.value, categoria.value]):
            ui.notify('Preencha todos os campos!', color='warning')
            return

        codigo_cidade = get_codigo_cidade(cidade.value)
        codigo_categoria = get_codigo_categoria(categoria.value)
        produto_valor = f"A{random.randint(1, 100):03}" if produto.value == 'Gerar Aleatório' else produto.value
        codigo_completo = f"{codigo_cidade}{empresa.value}{produto_valor}{codigo_categoria}"

        # Verifica se já existe no banco
        if session.query(Codigo).filter_by(codigo_completo=codigo_completo).first():
            ui.notify('Código já existe!', color='negative')
            return

        novo_codigo = Codigo(
            codigo_completo=codigo_completo,
            cidade=codigo_cidade,
            empresa=empresa.value,
            produto=produto_valor,
            categoria=codigo_categoria
        )
        session.add(novo_codigo)
        session.commit()
        ui.notify(f'Código gerado: {codigo_completo}', color='positive')
        atualizar_tabela()

    ui.button('Criar', on_click=criar).props('color=accent')
    atualizar_tabela()

ui.run()
