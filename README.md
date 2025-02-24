Tabela-Verdade-Mode-Easy--Projeto-Fatec-1-2025
import tkinter as tk
from tkinter import ttk, messagebox
import itertools
import re
import ttkbootstrap as tb  # Biblioteca para o visual despojado

# Dicionário para substituir operadores lógicos por equivalentes em Python
OPERADORES = {
    '¬': 'not ',
    '~': 'not ',
    '∧': ' and ',
    '^': ' and ',
    '∨': ' or ',
    '→': ' <= ',
    '↔': ' == '
}

def converter_expressao(expressao):
    """ Converte a expressão lógica para um formato compatível com eval(). """
    for simbolo, substituto in OPERADORES.items():
        expressao = expressao.replace(simbolo, substituto)
    return expressao

def avaliar_expressao(expressao, valores):
    """ Avalia a expressão lógica substituindo as variáveis pelos valores dados. """
    expressao_modificada = expressao
    for var, val in valores.items():
        expressao_modificada = re.sub(rf'\b{var}\b', str(val), expressao_modificada)

    expressao_modificada = converter_expressao(expressao_modificada)
    
    try:
        return eval(expressao_modificada)
    except Exception as e:
        resultado_label.config(text=f"Erro na expressão: {e}")
        return None

def gerar_tabela():
    expressao = entrada_expressao.get().strip().upper()

    # Extrair variáveis únicas e ordenar
    variaveis = sorted(set(re.findall(r'[A-Z]', expressao)))

    if not variaveis:
        resultado_label.config(text="Erro: Nenhuma variável encontrada!")
        return

    # Garante que apenas P, Q e R sejam consideradas
    todas_variaveis = ["P", "Q", "R"]
    variaveis = [var for var in todas_variaveis if var in variaveis]

    # Gerar todas as combinações possíveis de valores de verdade (2^n linhas)
    combinacoes = list(itertools.product([True, False], repeat=len(variaveis)))

    # Limpar a tabela antes de adicionar novos valores
    for i in tabela.get_children():
        tabela.delete(i)

    for combinacao in combinacoes:
        valores = dict(zip(variaveis, combinacao))
        resultado = avaliar_expressao(expressao, valores)

        if resultado is None:
            return  # Se houver erro na avaliação, interrompe a execução

        # Converter valores booleanos para "V" e "F"
        linha = [("V" if valores[var] else "F") for var in variaveis]
        linha.append("V" if resultado else "F")

        tabela.insert("", "end", values=linha)

    # Atualizar os cabeçalhos da tabela dinamicamente
    tabela["columns"] = variaveis + ["Resultado"]
    tabela["show"] = "headings"
    for var in variaveis:
        tabela.heading(var, text=var)
    tabela.heading("Resultado", text="Resultado")

def inserir_operador(operador):
    """ Insere operadores no campo de entrada ao clicar nos botões """
    entrada_expressao.insert(tk.END, operador)

def mostrar_ajuda():
    """ Exibe um pop-up com instruções de uso. """
    msg = (
        "📌 **Conectivos Lógicos**:\n"
        "¬ (Negação)  ¬P\n"
        "∧ (E)  P ∧ Q\n"
        "∨ (Ou)  P ∨ Q\n"
        "→ Condicional (Se,Então) P → Q\n"
        "↔ Bicondicional (Se e Somente) P ↔ Q\n\n"
        "✅ **Exemplo**: (P ∨ Q) → ¬R\n"
        "⏳ **Passo a Passo**:\n"
        "1️⃣ Digite a expressão\n"
        "2️⃣ Clique em 'Gerar Tabela'\n"
        "3️⃣ Veja os resultados calculados"
    )
    messagebox.showinfo("Ajuda - Como Usar", msg)

# Criando a interface gráfica com tema escuro
root = tb.Window(themename="superhero")  # 🔥 Interface escura moderna
root.title("🔍 Tabela Verdade Para Leigos/Matheus de Paula - Fatec 1/2025")
root.geometry("650x550")

frame = ttk.Frame(root, padding=10)
frame.pack(fill="both", expand=True)

# Label e campo de entrada para expressão lógica
lbl_instrucao = ttk.Label(frame, text="Digite a expressão:", font=("Arial", 12))
lbl_instrucao.pack(pady=5)

entrada_expressao = ttk.Entry(frame, width=50, font=("Arial", 14))
entrada_expressao.pack(pady=5)

# Frame para botões dos operadores
frame_botoes = ttk.Frame(frame)
frame_botoes.pack(pady=5)

operadores = ['¬', '∧', '∨', '→', '↔']
for op in operadores:
    btn = ttk.Button(frame_botoes, text=op, command=lambda op=op: inserir_operador(op), width=5, bootstyle="primary")
    btn.pack(side="left", padx=5)

# Botões principais
btn_gerar = ttk.Button(frame, text="🛠 Gerar Tabela Verdade", command=gerar_tabela, bootstyle="success-outline")
btn_gerar.pack(pady=5)

btn_ajuda = ttk.Button(frame, text="❓ Ajuda", command=mostrar_ajuda, bootstyle="info-outline")
btn_ajuda.pack(pady=5)

# Criando a tabela de saída
tabela_frame = ttk.Frame(frame)
tabela_frame.pack(fill="both", expand=True, pady=10)

tabela = ttk.Treeview(tabela_frame, show="headings")
tabela.pack(fill="both", expand=True)

# Label de erro ou sucesso
resultado_label = ttk.Label(frame, text="", foreground="red", font=("Arial", 12))
resultado_label.pack(pady=5)

root.mainloop()
