# FIAP - Faculdade de Informática e Administração Paulista

# CardioIA Fase 2: Diagnóstico Automatizado (IA no Estetoscópio Digital)

## 👨‍🎓 Integrante

- Pedro Gustavo França Moreira (RM: 568262)

## 👩‍🏫 Professores

- Tutor(a): Sabrina Otoni
- Coordenador(a): André Godoi

## 🎥 Vídeo de demonstração

**Link (YouTube, não listado):** COLAR_LINK_AQUI

---

## 📜 Descrição

Nesta fase o CardioIA ganha um módulo que lê relatos de pacientes escritos em linguagem natural e ajuda na triagem. São duas abordagens complementares:

1. **Parte 1, regras (NLP clássico):** um mapa de conhecimento liga expressões que o paciente usa ("aperto no tórax", "acordo sem ar") a possíveis doenças cardiovasculares. Um script em Python lê os relatos, encontra essas expressões e sugere o diagnóstico mais provável.
2. **Parte 2, Machine Learning:** uma base de 80 frases rotuladas como "alto risco" ou "baixo risco" é vetorizada com TF-IDF e usada para treinar classificadores do Scikit-learn, simulando um sistema de priorização em pronto atendimento.

Além do funcionamento, o projeto analisa **onde os modelos erram** e quais **vieses** aparecem, com uma reflexão de governança de dados.

> ⚠️ Projeto acadêmico com dados simulados. Não deve ser usado para diagnóstico ou triagem real.

---

## 📁 Estrutura

```
cardioia-fase2/
├── parte1/
│   ├── sintomas.txt               # 10 relatos simulados de pacientes
│   ├── mapa_conhecimento.csv      # sintoma_1 | sintoma_2 | doenca_associada (34 linhas, 7 doenças)
│   ├── parte1_extracao.ipynb      # leitura, extração de sintomas e sugestão de diagnóstico
│   └── resultados_parte1.csv      # saída gerada pelo notebook
├── parte2/
│   ├── frases_risco.csv           # 80 frases rotuladas (40 alto / 40 baixo risco)
│   └── parte2_classificador.ipynb # TF-IDF + Regressão Logística e Árvore de Decisão + avaliação
└── README.md
```

---

## 🩺 Parte 1: Extração de sintomas e sugestão de diagnóstico

**Como funciona:**

1. Texto normalizado (minúsculas e sem acento) para "tórax" e "torax" serem a mesma coisa.
2. O CSV vira uma ontologia `doença → conjunto de expressões`.
3. Cada expressão encontrada na frase é um voto para a doença. Uso regex com borda de palavra e um tratamento simples de **negação** ("não sinto dor no peito" não conta).
4. A doença com mais votos é sugerida, junto com uma confiança (votos da vencedora ÷ total) e as alternativas.

**Resultado:** os 10 relatos receberam um diagnóstico coerente com os sintomas descritos. Exemplos:

| Relato (resumo) | Sintomas encontrados | Sugestão |
|---|---|---|
| dor no peito que piora com esforço e passa no descanso | dor no peito, esforço físico, passa quando descanso | Angina (67%) |
| aperto no tórax indo pro braço esquerdo, suor frio, enjoo | 4 sintomas | Infarto (100%) |
| febre, dor que piora ao respirar e alivia inclinando | 4 sintomas | Pericardite (75%) |

**Limitação observada:** regras só reconhecem o que está escrito no mapa. "Meu peito tá doendo" não é identificado porque o mapa tem "dor no peito".

---

## 🤖 Parte 2: Classificador de risco

- **Base:** 80 frases simuladas e balanceadas (`frase,situacao`)
- **Pré-processamento:** minúsculas, sem acento, stopwords em português **mantendo "não", "sem" e "nunca"**
- **Vetorização:** TF-IDF com unigramas e bigramas, dentro de um `Pipeline` (evita vazamento de dados)
- **Modelos:** Regressão Logística e Árvore de Decisão
- **Divisão:** 75% treino / 25% teste estratificado + validação cruzada de 5 folds

| Modelo | Acurácia (teste) | Acurácia média (CV 5 folds) | Recall alto risco (teste) |
|---|---|---|---|
| Regressão Logística | 80% | ~79% | 70% |
| Árvore de Decisão | 80% | ~69% | 60% |

**Modelo escolhido:** Regressão Logística (generaliza melhor e é explicável pelos pesos das palavras).

### Vieses e distorções encontrados

- **Viés de palavra-chave:** "dor no peito de tanto rir no churrasco" → alto risco com mais de 90% de confiança.
- **Negação ignorada:** "não sinto dor no peito nem falta de ar" → alto risco.
- **Palavra fora do vocabulário:** "infarto" sozinho → baixo risco, porque não existe na base de treino.
- **Erro mais comum é o perigoso:** os modelos deixam passar mais casos graves (falso negativo) do que geram alarmes falsos.

### Governança

Base pequena, escrita e rotulada por uma pessoa sem formação médica, sem diversidade de jeitos de falar. Em saúde isso pode significar que pacientes com relatos atípicos (como mulheres com infarto sem dor no peito clássica) sejam subestimados. Um sistema assim só faz sentido como apoio ao profissional, com rótulos validados clinicamente e auditoria das decisões.

---

## 🔧 Como executar

```bash
git clone https://github.com/pedromoreira427/cardioia-fase2.git
cd cardioia-fase2
pip install pandas scikit-learn matplotlib jupyter
jupyter notebook
```

Abra os notebooks dentro de `parte1/` e `parte2/` e execute todas as células (eles leem os arquivos da própria pasta). Também funciona no Google Colab fazendo upload do notebook e do arquivo de dados correspondente.

---

## 🚀 Ir Além

- **Ir Além 1, portal React:** [pedromoreira-cardioia-portal](https://github.com/pedromoreira427/pedromoreira-cardioia-portal)
- **Ir Além 2, ECG com MLP:** [cardioia-ecg-mlp](https://github.com/pedromoreira427/cardioia-ecg-mlp)

## 📋 Licença

Projeto acadêmico desenvolvido para a FIAP, sob licença [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/).
