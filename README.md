# 🔎 ScriptSearcher

```
import requests

def buscar_scripts(busca, pagina=1):
    url = f"https://scriptblox.com/api/script/search?q={busca}&script%20name=5&page={pagina}"
    resposta = requests.get(url)
    dados = resposta.json()
    return dados.get('result', {}).get('scripts', []), dados.get('result', {}).get('totalPages', 1)

def mostrar_scripts(scripts):
    for script in scripts:
        print(f"Título: {script['title']}")
        print(f"Jogo: {script['game']['name']}")
        print(f"Tipo: {script['scriptType']}")
        print(f"Visualizações: {script['views']}")
        print(f"Criado: {script['createdAt']}")
        print(f"Atualizado: {script['updatedAt']}")
        print(f"Verificado: {'Sim' se script['verified'] senão 'Não'}")
        print(f"Chave Necessária: {'Sim' se script['key'] senão 'Não'}")
        print(f"Script: {script['script']}\n")

def principal():
    busca = input("Digite a busca: ")
    pagina = 1

    while True:
        scripts, total_paginas = buscar_scripts(busca, pagina)
        if scripts:
            mostrar_scripts(scripts)
            if pagina < total_paginas:
                mais = input("Carregar mais? (s/n): ").strip().lower()
                if mais == 's':
                    pagina += 1
                else:
                    break
            else:
                print("Nenhum script adicional encontrado.")
                break
        else:
            print("Nenhum script encontrado.")
            break

if __name__ == "__main__":
    principal()
