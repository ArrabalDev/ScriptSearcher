# 🔎 ScriptSearcher (Python)

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
```


# 🔍 ScriptSearcher (discord.py)
**Sistema de páginas adicionado!**
```
import discord
from discord.ext import commands
from discord import app_commands
import requests

intents = discord.Intents.all()
bot = commands.Bot(command_prefix="!", intents=intents)
cache = {}

@bot.event
async def on_ready():
    print(f'Logado como {bot.user}')
    await bot.tree.sync()

def buscar(term, pag=1):
    url = f"https://scriptblox.com/api/script/search?q={term}&script%20name=5&page={pag}"
    resp = requests.get(url)
    data = resp.json()
    return data.get('result', {}).get('scripts', []), data.get('result', {}).get('totalPages', 1)

async def auto(interaction: discord.Interaction, current: str):
    scripts, _ = buscar(current)
    cache[interaction.user.id] = scripts
    return [app_commands.Choice(name=script['title'], value=script['title']) for script in scripts[:25]]

@bot.tree.command(name="buscar", description="Busca scripts na ScriptBlox")
@app_commands.describe(term="Termo de busca para os scripts", pag="Número da página de resultados")
@app_commands.autocomplete(term=auto)
async def buscar_cmd(interaction: discord.Interaction, term: str, pag: int = 1):
    scripts, total_pages = buscar(term, pag)
    cache[interaction.user.id] = scripts
    
    if not scripts:
        await interaction.response.send_message("Nenhum script encontrado.", ephemeral=True)
        return

    embed = discord.Embed(title=f"(Página {pag} de {total_pages})", color=discord.Color.blue())
    for script in scripts:
        embed.add_field(name=script['title'], value=f"Jogo: {script['game']['name']} | Tipo: {script['scriptType']} | Visualizações: {script['views']}", inline=False)
    
    if total_pages > 1:
        embed.set_footer(text=f"Use /buscar term:{term} pag:<número> para navegar entre as páginas")

    await interaction.response.send_message(embed=embed, ephemeral=True)

bot.run('TOKEN')
```