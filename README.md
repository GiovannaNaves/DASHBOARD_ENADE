# Pipeline ENADE - Manutenção e Evolução

Este documento explica como **atualizar a base de dados** (incluir novos anos do ENADE), **rodar o pipeline em Python**, e **atualizar o painel no Power BI** a partir do resultado gerado. Também serve como referência para quem for dar manutenção ou evoluir o pipeline no futuro.

---

## 1. Estrutura de pastas necessária

O notebook Python (`Pipeline_ENADE_TCC_Giovanna_multianos.ipynb`) espera encontrar uma pasta `data` **ao lado dele**, com uma subpasta por ano:

```
TCC_ENADE/                              ← pasta qualquer
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
- O nome da subpasta deve ser **só o número do ano** (`2019`, `2021`, `2023`...). O pipeline detecta automaticamente todas as pastas com nome numérico dentro de `data/`.
- Dentro de cada pasta de ano, coloque **apenas os arquivos de microdados** (`microdados{ano}_arqN.csv`, baixados do site do INEP). **Não copie a pasta "Leia-me" / dicionário de dados que o INEP inclui no ZIP**, ela não é usada e pode ser descartada.

### Onde baixar os dados
Microdados do ENADE: https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enade
Baixe o ZIP do ano desejado, descompacte, e copie só os arquivos `microdados{ano}_arq*.csv` (ou `.txt`) para dentro de `data/{ano}/`.

---

## 2. Como rodar o pipeline em Python

Você precisa de um programa capaz de abrir e executar notebooks Jupyter (`.ipynb`). Duas opções:

### Opção A — VS Code (recomendado)
1. Instale o [Visual Studio Code](https://code.visualstudio.com/).
2. Instale o [Python](https://www.python.org/downloads/) (marque "Add Python to PATH" durante a instalação, no Windows).
3. Abra o VS Code → vá em **Extensões** (ícone de blocos no menu lateral) → instale **Python** e **Jupyter** (ambas da Microsoft).
4. No VS Code, **Abrir Pasta** → selecione a pasta `TCC_ENADE` (a que contém o notebook e a pasta `data`).
5. Clique no arquivo `Pipeline_ENADE_TCC_Giovanna_multianos.ipynb` para abri-lo.
6. No topo do notebook, clique em **"Executar Tudo"** (Run All) — ícone de "▶▶" na barra de ferramentas do notebook.
7. Na primeira execução, o VS Code pode pedir para selecionar um "kernel" (interpretador Python) — escolha a versão do Python instalada.

### Opção B — Anaconda / Jupyter Notebook
1. Instale o [Anaconda](https://www.anaconda.com/download) (já vem com Python e Jupyter).
2. Abra o **Anaconda Navigator** → inicie o **Jupyter Notebook** (ou Jupyter Lab).
3. Na janela do navegador que abrir, navegue até a pasta `TCC_ENADE` e clique no arquivo `.ipynb`.
4. No menu do notebook: **Cell → Run All** (ou **Kernel → Restart & Run All**, para garantir que tudo rode do zero).

### O que esperar durante a execução
- A primeira célula instala algumas bibliotecas (`pip install ...`), é normal levar alguns segundos.
- O notebook processa **um ano por vez**. Para bases grandes (muitos anos, arquivos grandes), pode levar alguns minutos.
- Se faltar algum arquivo `arqN` em algum ano, você verá um aviso amarelo (`⚠️ Arquivos não encontrados`), não é um erro, o pipeline continua e só não preenche as colunas daquele arquivo específico.
- Se um ano inteiro falhar (ex.: pasta vazia), o pipeline mostra `Falha no ano {ano}` e **continua para o próximo ano**, não trava o processamento dos demais.

### Onde aparece o resultado
Ao final, uma pasta **`outputs/`** é criada automaticamente (se não existir) dentro da mesma pasta do notebook:

```
TCC_ENADE/
├── Pipeline_ENADE_TCC_Giovanna_multianos.ipynb
├── data/
└── outputs/                                    ← criada automaticamente
    ├── dashboard_export_2019.csv                (um por ano)
    ├── dashboard_export_2021.csv
    ├── dashboard_export_2023.csv
    ├── dashboard_consolidado_multianos.csv      ← ESTE é o arquivo usado no Power BI
    ├── enade_curso_2019.parquet                 (backup completo, não usado no Power BI)
    └── ... (gráficos .png e CSVs de correlação/cluster)
```

**O arquivo que importa para o Power BI é sempre `outputs/dashboard_consolidado_multianos.csv`**, ele já reúne todos os anos processados em uma única tabela (1 linha = 1 curso em 1 ano).

---

## 3. Como atualizar o Power BI com o novo arquivo

Depois de gerar um novo `dashboard_consolidado_multianos.csv` (com mais anos, ou dados atualizados), não é preciso refazer o relatório do zero, basta apontar a consulta original para o novo arquivo:

1. Abra o relatório no Power BI → vá para o **conjunto de dados / modelo semântico** → **Transformar dados** (abre o Power Query).
2. Na lista de consultas à esquerda, selecione a consulta **`dashboard_consolidado_multianos`** (a tabela mestre/original, não as tabelas `*_long`).
3. No painel **Passos Aplicados**, clique no ícone de **engrenagem ⚙️** ao lado do primeiro passo, **"Origem"**.
4. Na janela que abrir, clique em **Procurar** e selecione o novo arquivo CSV (mesmo nome ou não, só precisa ter a mesma estrutura de colunas).
5. Confirme. O Power Query tenta reaplicar todos os passos seguintes (tipos de coluna, criação da `Chave`, filtros de nulos, etc.) sobre o novo arquivo.
6. Confira a lista de **Passos Aplicados**: se algum passo aparecer com um aviso, clique nele para ver o que quebrou (geralmente indica que uma coluna esperada não existe mais ou mudou de nome — só ocorre se a estrutura do CSV for alterada).
7. Clique em **Guardar** / **Aplicar**.
8. As tabelas `*_long` (Raça, Sexo, Renda, etc.) são **referências** da tabela mestre, então atualizam automaticamente, não precisa repetir nada nelas.
9. No relatório, clique em **Atualizar** (Refresh) na barra de ferramentas para os gráficos recarregarem com os dados novos.

> **Dica:** para já deixar isso ainda mais simples no futuro, salve o `dashboard_consolidado_multianos.csv` sempre no **mesmo caminho/nome de arquivo** (por exemplo, sempre sobrescrevendo o arquivo anterior na mesma pasta). Assim, no passo 4, o Power Query já reconhece o caminho automaticamente e às vezes nem pede para selecionar o arquivo de novo, só clicar em **Atualizar**.

---

## 4. Conceito importante para manter ao evoluir o pipeline

**Cada linha da base final representa UM CURSO em UM ANO, nunca um aluno.**

O ENADE não fornece um identificador de aluno em comum entre os diferentes arquivos de microdados (`arq1`, `arq3`, `arq5`...) de um mesmo ano. A linha X de um arquivo **não** é necessariamente o mesmo estudante que a linha X de outro arquivo. Por isso o pipeline:

1. Lê cada arquivo separadamente;
2. Resume (agrega) cada arquivo por curso **antes** de juntar com qualquer outro arquivo, calculando médias, modas ou contagens por categoria;
3. Só então junta essas agregações entre si, usando `CO_CURSO` como chave (já no nível de curso, nunca de aluno).

**Ao manter ou evoluir este pipeline, nunca tente "alinhar" linhas de arquivos diferentes pela posição (ex.: linha 5 de um arquivo = linha 5 de outro), isso reintroduziria um erro metodológico já corrigido neste projeto.** Qualquer nova variável deve seguir o mesmo padrão: agregar primeiro, juntar depois.

---

## 5. [ADICIONAL] Como adicionar uma nova variável do questionário (evolução do pipeline)

Toda a configuração fica concentrada na célula de configuração do notebook (variáveis em letras maiúsculas, no início). Para adicionar uma nova pergunta do questionário (ex.: `QE_I06`):

1. **`ARQUIVO_VARIAVEIS`**: adicione a variável na entrada do arquivo onde ela está (ex.: `"arq12": ["QE_I06"]`, conforme a tabela de variáveis do INEP).
2. Decida o tipo de agregação e adicione o nome da variável na lista correspondente:
   - **Numérica/ordinal** (média por curso, ex. nota, idade) → `VARS_NUMERICAS_MEDIA`
   - **Categórica** (contagem absoluta + proporção por categoria, ex. sexo, raça, renda) → `VARS_CATEGORICAS_PROPORCAO`
   - **Atributo do curso, praticamente constante** (ex. UF, modalidade) → `VARS_IDENTIDADE_CURSO`
3. Se for categórica, adicione os nomes amigáveis das colunas de contagem no dicionário **`NOMES_QTD`** (uma entrada por categoria/código, ex. `"A": "QTD_NOVAVAR_CATEGORIA1"`), sem isso, a coluna ainda é criada, mas com um nome genérico (`QTD_QE_I06_A`).
4. Se quiser um rótulo legível para gráficos/relatórios, adicione também em **`LABELS`**.
5. Rode o notebook (Executar Tudo), a nova variável passa a aparecer automaticamente no dashboard (todas as colunas `QTD_*` são incluídas dinamicamente, não é preciso editar a lista de colunas do dashboard manualmente).

Para adicionar um **novo ano**, basta criar a pasta `data/{ano}/` com os arquivos, nenhuma alteração de código é necessária.

---

## 6. Problemas comuns

**"A coluna 'Chave' contém um valor duplicado" ao criar relação no Power BI**
Geralmente indica linhas com `NU_ANO` ou `CO_CURSO` vazios no CSV (ex.: linha em branco no final do arquivo). No Power Query, filtre essas linhas vazias antes do passo que cria a coluna `Chave` (veja seção 3 e histórico de suporte do projeto para o passo a passo).

**Números de nota aparecem gigantes/estranhos no Excel (ex.: `4.452.727.272.727.270`)**
Os CSVs exportados pelo pipeline já usam `;` como separador e `,` como decimal (formato Excel-BR) e os valores são arredondados antes de exportar, então isso não deve mais ocorrer com os arquivos gerados por este pipeline. Se acontecer com um CSV editado manualmente depois, verifique se ele não acumulou casas decimais demais ao ser salvo novamente pelo Excel.

**Quero exportar uma variável categórica já no formato "longo" (uma linha por categoria), sem precisar fazer "Anular Dinamização" no Power Query**
Isso é possível adicionando uma exportação extra ao pipeline Python, fale com quem mantém o notebook para incluir essa função, caso o volume de manutenção no Power BI fique grande demais.