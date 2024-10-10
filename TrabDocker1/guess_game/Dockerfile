# Use uma imagem base Python
FROM python:3.8

# Define o diretório de trabalho
WORKDIR /app

# Copia os arquivos necessários
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copia o restante do código
COPY . .

# Expõe a porta em que a aplicação Flask escutará
EXPOSE 4000

# Comando para iniciar a aplicação Flask
CMD ["bash", "start-backend.sh"]
