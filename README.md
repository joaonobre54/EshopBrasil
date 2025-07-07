# EshopBrasil# ShopBrasil - Sistema CRUD com MongoDB e Streamlit

## 💡 Introdução

O projeto **ShopBrasil** visa demonstrar o uso integrado do MongoDB e do Streamlit para criar uma aplicação de gestão de clientes, produtos e pedidos em um ambiente de comércio eletrônico fictício.

## 🎯 Objetivos do Projeto

- Criar um ambiente padronizado e isolado usando Docker.
- Desenvolver uma aplicação web interativa com Streamlit.
- Gerenciar dados de clientes, produtos e pedidos de forma prática.

## 🛠️ Descrição do Projeto

### Docker

Utilizamos **Docker** para criar um ambiente isolado e facilitar a replicação do projeto. A configuração inclui um container para o MongoDB.

### MongoDB

Um container MongoDB é utilizado para armazenar os dados, simulando as coleções `clientes`, `produtos` e `pedidos`.

### Streamlit

A aplicação em `app.py` permite:

- Conectar ao MongoDB.
- Inserir, editar e excluir dados.
- Concatenar dados de diferentes coleções.
- Realizar consultas e exibir resultados na interface gráfica.

## 🚀 Passos para Implementação

### Pré-requisitos

- Docker e Docker Compose instalados
- Python 3.9 ou superior

### Clonar o repositório

```bash
git clone https://github.com/seu-usuario/ShopBrasil-Projeto.git
cd ShopBrasil-Projeto
docker-compose up -d
pip install -r requirements.txt
streamlit run app.py
CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    telefone VARCHAR(20),
    endereco TEXT,
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    categoria VARCHAR(100),
    preco DECIMAL(10,2) NOT NULL,
    estoque INT NOT NULL,
    descricao TEXT,
    data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INT REFERENCES clientes(id),
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total DECIMAL(10,2),
    status VARCHAR(50) DEFAULT 'Pendente'
);

CREATE TABLE itens_pedido (
    id SERIAL PRIMARY KEY,
    pedido_id INT REFERENCES pedidos(id),
    produto_id INT REFERENCES produtos(id),
    quantidade INT NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL
);

---

# 📄 **docker-compose.yml**

```yaml
version: '3.8'

services:
  mongo:
    image: mongo:latest
    container_name: mongo
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_DATABASE: shopbrasil
streamlit
pymongo
import streamlit as st
from pymongo import MongoClient
from datetime import datetime

# Conexão com MongoDB
client = MongoClient("mongodb://localhost:27017/")
db = client["shopbrasil"]

st.title("ShopBrasil - Sistema CRUD")

menu = ["Inserir", "Editar/Excluir", "Consultar"]
choice = st.sidebar.selectbox("Menu", menu)

clientes = db["clientes"]
produtos = db["produtos"]
pedidos = db["pedidos"]

if choice == "Inserir":
    st.subheader("Inserir Cliente")
    nome = st.text_input("Nome")
    email = st.text_input("Email")
    telefone = st.text_input("Telefone")
    endereco = st.text_area("Endereço")

    if st.button("Salvar Cliente"):
        clientes.insert_one({
            "nome": nome,
            "email": email,
            "telefone": telefone,
            "endereco": endereco,
            "data_cadastro": datetime.now()
        })
        st.success("Cliente inserido com sucesso!")

    st.subheader("Inserir Produto")
    nome_prod = st.text_input("Nome do Produto")
    categoria = st.text_input("Categoria")
    preco = st.number_input("Preço", 0.0)
    estoque = st.number_input("Estoque", 0)
    descricao = st.text_area("Descrição")

    if st.button("Salvar Produto"):
        produtos.insert_one({
            "nome": nome_prod,
            "categoria": categoria,
            "preco": preco,
            "estoque": estoque,
            "descricao": descricao,
            "data_criacao": datetime.now()
        })
        st.success("Produto inserido com sucesso!")

elif choice == "Editar/Excluir":
    st.subheader("Clientes")
    all_clients = list(clientes.find())
    for c in all_clients:
        st.write(f"{c['nome']} | {c['email']}")
        if st.button(f"Excluir {c['nome']}"):
            clientes.delete_one({"_id": c["_id"]})
            st.success("Cliente excluído com sucesso!")

elif choice == "Consultar":
    st.subheader("Clientes Cadastrados")
    for c in clientes.find():
        st.json(c)

    st.subheader("Produtos Cadastrados")
    for p in produtos.find():
        st.json(p)

    st.subheader("Concatenar Dados")
    all_clients = list(clientes.find())
    all_products = list(produtos.find())

    for cl in all_clients:
        for pr in all_products:
            st.write(f"Cliente: {cl['nome']} poderia comprar: {pr['nome']}")
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["streamlit", "run", "app.py"]

