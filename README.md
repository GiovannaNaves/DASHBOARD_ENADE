# Pipeline ENADE - Manutenção e Evolução

Este documento explica como **atualizar a base de dados** (incluir novos anos do ENADE), **rodar o pipeline em Python**, e **atualizar o painel no Power BI** a partir do resultado gerado.

---

## 1. Estrutura de pastas necessária

O notebook Python (`Pipeline_ENADE_TCC_Giovanna_multianos.ipynb`) espera encontrar uma pasta `data` **ao lado dele**, com uma subpasta por ano:

```
TCC_ENADE/                        ← pasta qualquer
├── Pipeline_ENADE_TCC_Giovanna_multianos.ipynb
└── data/
    ├── 2019/
    │   ├── microdados2019_arq1.csv
    │   ├── microdados2019_arq3.csv
    │   ├── microdados2019_arq5.csv
    │   └── ... (demais arquivos arqN daquele ano)
    ├── 2021/
    │   ├── microdados2021_arq1.csv
    │   └── ...
    └── 2023/
        └── ...
```

**Importante:**
- O nome da subpasta deve ser **apenas o número do ano** (`2019`, `2021`, `2023`...). O pipeline detecta automaticamente todas as pastas com nome numérico dentro de `data/`.
- Dentro de cada pasta de ano, coloque **apenas os arquivos de microdados** (`microdados{ano}_arqN.csv`, baixados do site do INEP). **Não copie a pasta "Leia-me" / dicionário de dados que o INEP inclui no ZIP**, ela não é usada e pode ser descartada.

### Origem dos dados
Microdados do ENADE: https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enade
Baixe o ZIP do ano desejado, descompacte, e copie só os arquivos `microdados{ano}_arq*.csv` (ou `.txt`) para dentro de `data/{ano}/`.

---

## 2. Como rodar o pipeline em Python

É necessária a instalação de um programa capaz de abrir e executar notebooks Jupyter (`.ipynb`). Duas opções:

### Opção A - VS Code
1. Instale o [Visual Studio Code](https://code.visualstudio.com/).
2. Instale o [Python](https://www.python.org/downloads/) (marque "Add Python to PATH" durante a instalação, no Windows).
3. Abra o VS Code → vá em **Extensões** (ícone de blocos no menu lateral) → instale **Python** e **Jupyter** (ambas da Microsoft).
4. No VS Code, **Abrir Pasta** → selecione a pasta `TCC_ENADE` (a que contém o notebook e a pasta `data`).
5. Clique no arquivo `Pipeline_ENADE_TCC_Giovanna_multianos.ipynb` para abri-lo.
6. No topo do notebook, clique em **"Executar Tudo"** (Run All), ícone de "▶▶" na barra de ferramentas do notebook.
7. Na primeira execução, o VS Code pode pedir para selecionar um "kernel" (interpretador Python), escolha a versão do Python instalada.

### Opção B - Anaconda / Jupyter Notebook
1. Instale o [Anaconda](https://www.anaconda.com/download) (já vem com Python e Jupyter).
2. Abra o **Anaconda Navigator** → inicie o **Jupyter Notebook** (ou Jupyter Lab).
3. Na janela do navegador que abrir, navegue até a pasta `TCC_ENADE` e clique no arquivo `.ipynb`.
4. No menu do notebook: **Cell → Run All** (ou **Kernel → Restart & Run All**).


### Resultado
Ao final, uma pasta **`outputs/`** é criada automaticamente (se não existir) dentro da mesma pasta do notebook:

```
TCC_ENADE/
├── Pipeline_ENADE_TCC_Giovanna_multianos.ipynb
├── data/
└── outputs/              ← criada automaticamente
      └── dashboard_consolidado_multianos.csv   
```

**O arquivo que importa para o Power BI é `outputs/dashboard_consolidado_multianos.csv`**.

---

## 3. Como atualizar o Power BI com o novo arquivo

Depois de gerar um novo `dashboard_consolidado_multianos.csv` (com mais anos, ou dados atualizados), não é preciso refazer o relatório, basta apontar a consulta para o novo arquivo:

1. Abra o relatório no Power BI → vá para o **conjunto de dados / modelo semântico** → **Transformar dados** (abre o Power Query).
2. Na lista de consultas à esquerda, selecione a consulta **`dashboard_consolidado_multianos`** (a tabela mestre/original, não as tabelas `*_long`).
3. No painel **Passos Aplicados**, clique no ícone de **engrenagem** ao lado do primeiro passo, **"Origem"**.
4. Na janela que abrir, clique em **Procurar** e selecione o novo arquivo CSV.
5. Confirme. O Power Query tenta reaplicar todos os passos seguintes sobre o novo arquivo.
6. Confira a lista de **Passos Aplicados**: se algum passo aparecer com um aviso, clique nele para ver o que quebrou (geralmente indica que uma coluna esperada não existe mais ou mudou de nome, só ocorre se a estrutura do CSV for alterada).
7. Clique em **Guardar** / **Aplicar**.
8. No relatório, clique em **Atualizar** (Refresh) na barra de ferramentas para os gráficos recarregarem com os dados novos.

> Para facilitar, salve o `dashboard_consolidado_multianos.csv` sempre no **mesmo caminho/nome de arquivo** (por exemplo, sempre sobrescrevendo o arquivo anterior na mesma pasta). Assim, no passo 4, o Power Query já reconhece o caminho automaticamente e basta clicar em **Atualizar**.

---
