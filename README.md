# Conexão as APIs
# Gerar relatório

print("-- Iniciando o Processo --")

# Biblioteca requições WEB
import requests
# Biblioteca JSON
import json

# URLs/Endereços
URL_USUARIOS = "https://jsonplaceholder.typicode.com/users"
URL_TAREFAS = "https://jsonplaceholder.typicode.com/todos"
URL_POSTAGENS = "https://jsonplaceholder.typicode.com/posts"

# -- 1- Dados dos usuários --
print("\nPasso 1: Lista de usuários...")
try:
    # Busca de dados
    resposta_usuarios = requests.get(URL_USUARIOS, timeout=5)
    # Erro na busca
    resposta_usuarios.raise_for_status()
    # Python
    usuarios = resposta_usuarios.json()
    print("Usuários encontrados com sucesso.")
except requests.exceptions.RequestException as e:
    print(f"Erro ao buscar usuários: {e}")
    # Em caso de erro, é finalizado aqui
    exit()

# -- 2-  Dados das tarefas --
print("\nPasso 2: Buscando lista de tarefas...")
try:
    resposta_tarefas = requests.get(URL_TAREFAS, timeout=5)
    resposta_tarefas.raise_for_status()
    tarefas = resposta_tarefas.json()
    print("Tarefas encontradas com sucesso.")
except requests.exceptions.RequestException as e:
    print(f"Erro ao buscar tarefas: {e}")
    exit()

# -- 3- Dados para o relatório --
print("\nPasso 3: Calculando tarefas concluídas e pendentes por usuário...")

# Dicionário vazio para guardar as contagens.
# As "chaves" são os IDs dos usuários e os "valores" são outros dicionários
# com as contagens de tarefas.
contagem_tarefas_por_usuario = {}

# Percorrendo a lista de tarefas, uma por uma
for tarefa in tarefas:
    # Pegar o ID do usuário dono da tarefa
    id_usuario = tarefa['userId']
    # Checamos se a tarefa foi completada
    esta_completa = tarefa['completed']

    # Se o ID do usuário não estiver no dicionário de contagem,
    # O código diciona ele e começa a contagem do zero.
    if id_usuario not in contagem_tarefas_por_usuario:
        contagem_tarefas_por_usuario[id_usuario] = {
            'completas': 0,
            'pendentes': 0
        }

    # +1 à contagem correta
    if esta_completa:
        contagem_tarefas_por_usuario[id_usuario]['completas'] += 1
    else:
        contagem_tarefas_por_usuario[id_usuario]['pendentes'] += 1

print("Cálculo finalizado.")

# -- 4- Lista de relatórios no desejado --
print("\nPasso 4: Relatório final...")

# Lista vazia para guardar os dicionários do relatório
relatorio_final = []

# Percorrer lista de usuários para criar o relatório
for usuario in usuarios:
    # Pegar o ID e o nome do usuário
    id_usuario = usuario['id']
    nome_usuario = usuario['name']

    # Contagem calculada no passo anterior
    contagens = contagem_tarefas_por_usuario.get(id_usuario, {'completas': 0, 'pendentes': 0})
    tarefas_completas = contagens['completas']
    tarefas_pendentes = contagens['pendentes']

    # Dicionário para este usuário, no formato que a API de postagens espera
    item_relatorio = {
        "userId": id_usuario,
        "title": f"Relatório de tarefas de {nome_usuario}",
        "body": f"Tarefas completas: {tarefas_completas}, Tarefas pendentes: {tarefas_pendentes}"
    }

    # Adiciona o dicionário à nossa lista de relatórios
    relatorio_final.append(item_relatorio)

print("Relatório pronto!")
# Exibição do Relatório em tela
print(json.dumps(relatorio_final, indent=4, ensure_ascii=False))

# -- 5- Enviando o relatório para a API --
print("\nPasso 5: Enviando o relatório via POST para a API...")

try:   
    # Se o relatório não estiver vazio, ir ao primeiro item
    if relatorio_final:
        primeiro_item = relatorio_final[0]
        # Envio dos dados para a URL de postagens
        resposta_post = requests.post(URL_POSTAGENS, json=primeiro_item, timeout=5)
        resposta_post.raise_for_status()
        print("Relatório enviado com sucesso para a API.")
        print(f"Resposta da API (status code): {resposta_post.status_code}")
        # print("Resposta da API:")
        # print(resposta_post.json())
    else:
        print("Relatório vazio, nada para enviar.")

except requests.exceptions.RequestException as e:
    print(f"Erro ao enviar o relatório: {e}")

print("\n--- Processo concluído! ---")# TesteVagaPrograma-o
Teste para vaga interna
