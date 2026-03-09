# Temporal Modeling and Counterfactual Policy Simulation of Student Dropout

Notebook principal do projeto que dá suporte ao paper **“A Mathematical Framework for Temporal Modeling and Counterfactual Policy Simulation of Student Dropout”**. O pipeline transforma dados temporais de engajamento acadêmico em uma base semanal no formato *person-period*, treina um modelo de risco temporal, reconstrói trajetórias de sobrevivência e executa simulações contrafactuais de política para comparar cenários de intervenção. fileciteturn1file0

## Objetivo

Este repositório foi organizado para reproduzir, auditar e estender o experimento central do paper. O foco não é apenas prever **quem** tem maior risco de evasão, mas também **quando** o risco aumenta ao longo do tempo acadêmico e como regras explícitas de intervenção alteram, em simulação, as trajetórias de sobrevivência estimadas pelo modelo. fileciteturn1file1

## O que este notebook faz

O notebook implementa um fluxo fim a fim que cobre:

1. integração das tabelas do **OULAD**;
2. construção da unidade de análise no nível de **enrollment**;
3. expansão semanal em formato **person-period**;
4. definição do evento principal como **Withdrawn** com `date_unregistration` válido;
5. engenharia de covariáveis temporais, como `total_clicks`, `recency`, `streak` e indicadores de atividade;
6. divisão temporal estratificada com controle de vazamento;
7. treino do modelo principal de **discrete-time hazard** com regressão logística penalizada e calibração sigmoidal;
8. reconstrução das curvas de sobrevivência previstas;
9. simulação de políticas contrafactuais nas versões **shock** e **mechanism-aware**;
10. análise por subgrupos com *bootstrap* para medir mudança de gap entre grupos observáveis. fileciteturn1file2 fileciteturn1file3

## Perguntas de pesquisa cobertas

O notebook foi estruturado para responder três perguntas centrais:

- **RQ1 — Qualidade do hazard temporal:** o modelo semanal discrimina e calibra bem o suficiente para sustentar decisões downstream?
- **RQ2 — Contraste estrutural entre regimes:** uma regra determinística baseada em recência produz contraste interpretável de sobrevivência sob diferentes cenários simulados?
- **RQ3 — Mudança de gap entre subgrupos:** a mesma política altera o gap entre grupos observáveis com direção estável sob incerteza por *bootstrap*? fileciteturn1file1 fileciteturn1file2

## Resumo metodológico

### 1) Unidade de análise

A unidade analítica é o **enrollment**, identificado pelo triplo:

```text
(id_student, code_module, code_presentation)
```

Após deduplicação, o paper reporta **32.593 enrollments** e **28.785 estudantes únicos**. O evento principal exige `final_result = Withdrawn` e `date_unregistration` válido; casos Withdrawn sem data válida são tratados como censurados sob a definição principal do endpoint. fileciteturn1file2

### 2) Representação temporal

Cada enrollment é expandido em semanas, formando uma tabela **person-period** com uma linha por enrollment-semana. Isso permite modelar o risco como um processo temporal, em vez de uma classificação estática de fim de curso. fileciteturn1file1

### 3) Modelo principal

O backbone do trabalho é um modelo de **discrete-time hazard** ajustado sobre linhas semanais, com regressão logística balanceada e calibração posterior. A partir do hazard semanal previsto, o notebook reconstrói a sobrevivência por produto cumulativo. fileciteturn1file0 fileciteturn1file3

### 4) Política contrafactual

O notebook compara dois tipos de regime simulado:

- **Shock:** reduz diretamente o hazard dentro da janela ativa.
- **Mechanism-aware:** altera covariáveis contrafactuais e recalcula o risco de forma stateful. fileciteturn1file1 fileciteturn1file3

### 5) Horizonte de avaliação

O protocolo separa três horizontes:

- `Tpolicy = 18`: horizonte substantivo principal;
- `Teval_policy = 38`: suporte bruto de trajetória para política;
- `Teval_metrics = 37`: horizonte estável para métricas ponderadas por IPCW. fileciteturn1file4

## Estrutura esperada do repositório

```text
.
├── notebooks/
│   └── main_notebook.ipynb
├── outputs_v2/
│   ├── figures/
│   └── tables/
├── data/
│   └── OULAD files
├── requirements.txt
└── README.md
```

> Ajuste os nomes de pastas e arquivos conforme a estrutura real do seu repositório.

## Principais artefatos exportados

Além do notebook, o projeto exporta artefatos em `outputs_v2` para dar rastreabilidade às afirmações do paper. Entre os arquivos destacados no manuscrito estão:

- `table_policy_spec.csv`
- `table_policy_scenarios_main.csv`
- `table_policy_scenario_params.csv`
- `table_policy_deltaS_by_week_by_scenario.csv`
- `table_policy_deltaS_at_horizons_by_scenario.csv`
- `table_policy_horizons_dual.csv`
- `table_policy_mech_operator_spec.csv`
- `table_rq2_sensitivity_grid.csv` fileciteturn1file4

Esses artefatos documentam o contrato de política, os cenários, os horizontes de avaliação e os resultados exportados da simulação. fileciteturn1file4

## Como executar

### 1) Criar ambiente

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate   # Windows
pip install -r requirements.txt
```

### 2) Obter os dados

Baixe e organize as tabelas do **OULAD** na pasta de dados definida no notebook. O paper utiliza as tabelas de interação no VLE, avaliações e registros administrativos para montar a base temporal. fileciteturn1file2

### 3) Executar o notebook

Abra o notebook principal em Jupyter:

```bash
jupyter lab
```

Em seguida, execute as células na ordem original do pipeline, desde a construção do backbone até a exportação dos artefatos finais.

## Saídas analíticas esperadas

Ao final da execução, o notebook deve produzir:

- métricas de discriminação do hazard semanal;
- curvas de calibração;
- trajetórias médias de sobrevivência;
- curvas de `ΔS(t)` por cenário de política;
- diagnósticos de censoring;
- comparação com benchmarks e análises de robustez;
- análise de subgrupos com intervalos via *bootstrap*. fileciteturn1file3

## Escopo interpretativo

Os resultados de política e fairness **não** devem ser lidos como efeitos causais identificados. O próprio paper delimita que `ΔS` e `ΔGap` são **contrastes estruturais simulados**, dependentes do modelo ajustado e do contrato explícito da política, não estimativas causais observacionais. fileciteturn1file4

## Reprodutibilidade

Este repositório existe para servir como trilha executável do paper. O manuscrito descreve explicitamente que o código cobre:

- pré-processamento;
- split temporal;
- treino e calibração;
- simulação de política;
- análise de subgrupos;
- exportação dos artefatos em `outputs_v2`. fileciteturn1file4

Para manter a reprodutibilidade:

- não altere a lógica do endpoint sem atualizar o texto do paper;
- preserve os horizontes `Tpolicy`, `Teval_policy` e `Teval_metrics`;
- mantenha a nomenclatura dos artefatos exportados;
- registre qualquer mudança metodológica no notebook e no manuscrito.

## Citação

Se este repositório apoiar seu trabalho, cite o paper correspondente:

```bibtex
@article{daSilva2026temporal_dropout,
  title={A Mathematical Framework for Temporal Modeling and Counterfactual Policy Simulation of Student Dropout},
  author={da Silva, Rafael and Eicher, Jeff and Longo, Gregory},
  journal={Working paper / manuscript},
  year={2026}
}
```

## Observação

Este README foi escrito para o **notebook principal** que sustenta o paper. Caso o repositório tenha múltiplos notebooks auxiliares, vale adicionar uma seção extra descrevendo o papel de cada um.
