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

# 🔍 ScriptSearcher (discord.py)

```
import discord
from discord.ext import commands
from discord import app_commands
import requests

intents = discord.Intents.all()
bot = commands.Bot(command_prefix="!", intents=intents)
script_cache = {}

@bot.event
async def on_ready():
    print(f'Logado como {bot.user}')
    await bot.tree.sync()

def buscar(busca, pagina=1):
    url = f"https://scriptblox.com/api/script/search?q={busca}&script%20name=5&page={pagina}"
    resposta = requests.get(url)
    dados = resposta.json()
    return dados.get('result', {}).get('scripts', []), dados.get('result', {}).get('totalPages', 1)

async def autocompletar(interaction: discord.Interaction, current: str):
    scripts, _ = buscar(current)
    script_cache[interaction.user.id] = scripts
    return [app_commands.Choice(name=script['title'], value=script['title']) for script in scripts[:25]]

@bot.tree.command(name="buscar_scripts", description="Busca scripts na ScriptBlox")
@app_commands.describe(busca="Termo de busca para os scripts")
@app_commands.autocomplete(busca=autocompletar)
async def buscar_cmd(interaction: discord.Interaction, busca: str):
    scripts = script_cache.get(interaction.user.id, [])
    script = next((s for s in scripts if s['title'].lower() == busca.lower()), None)
    
    if script:
        embed = discord.Embed(title=script['title'], color=discord.Color.blue())
        embed.add_field(name="Jogo", value=script['game']['name'], inline=False)
        embed.add_field(name="Tipo", value=script['scriptType'], inline=False)
        embed.add_field(name="Visualizações", value=script['views'], inline=False)
        embed.add_field(name="Criado", value=script['createdAt'], inline=False)
        embed.add_field(name="Atualizado", value=script['updatedAt'], inline=False)
        embed.add_field(name="Verificado", value="Sim" if script['verified'] else "Não", inline=False)
        embed.add_field(name="Chave Necessária", value="Sim" if script['key'] else "Não", inline=False)
        embed.add_field(name="Script", value=f"```{script['script']}```", inline=False)
        await interaction.response.send_message(embed=embed, ephemeral=True)
    else:
        if scripts:
            options = [discord.SelectOption(label=s['title'], description=s['game']['name'], value=str(i)) for i, s in enumerate(scripts[:25])]
            select = discord.ui.Select(placeholder="Selecione um script...", options=options)

            async def select_callback(interaction: discord.Interaction):
                index = int(select.values[0])
                script = scripts[index]
                embed = discord.Embed(title=script['title'], color=discord.Color.blue())
                embed.add_field(name="Jogo", value=script['game']['name'], inline=False)
                embed.add_field(name="Tipo", value=script['scriptType'], inline=False)
                embed.add_field(name="Visualizações", value=script['views'], inline=False)
                embed.add_field(name="Criado", value=script['createdAt'], inline=False)
                embed.add_field(name="Atualizado", value=script['updatedAt'], inline=False)
                embed.add_field(name="Verificado", value="Sim" if script['verified'] else "Não", inline=False)
                embed.add_field(name="Chave Necessária", value="Sim" if script['key'] else "Não", inline=False)
                embed.add_field(name="Script", value=f"```{script['script']}```", inline=False)
                await interaction.response.send_message(embed=embed, ephemeral=True)

            select.callback = select_callback
            view = discord.ui.View()
            view.add_item(select)
            await interaction.response.send_message("Selecione um script para ver mais detalhes:", view=view, ephemeral=True)
        else:
            await interaction.response.send_message("Nenhum script encontrado.", ephemeral=True)

bot.run('TOKEN')