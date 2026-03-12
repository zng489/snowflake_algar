# Padronização de Nomes - Tasks e Tabelas (DWDEV.DATASCIENCE)

**Data da documentação:** Março 2026  
**Objetivo:** Uniformizar nomenclatura de tasks e tabelas no Snowflake para facilitar manutenção, rastreabilidade e governança de dados no time de Data Science.  
**Padrão base:** `TK_DS_<PROJETO>_<SUFIXO>` (para tasks)  
**Extensão para tabelas:** Sugestão de alinhamento usando mesmos projetos + sufixos, com ou sem prefixo `TK_DS_`.

## 1. Padrão de Nomenclatura para Tasks

Formato:  
**`TK_DS_<PROJETO>_<SUFIXO>`**

- **Prefixo fixo:** `TK_DS`  
- **<PROJETO>**: Identificador do tema/área (em maiúsculas, snake_case, sem acentos)  
  Exemplos: `CHURN_INADIMPLENCIA_CNPJ`, `MAU_PAGADOR_CPF`, `VISAO_CLIENTE_B2B`, `RISCO_CLIENTE_CORP`, `PROPENSAO_NOMO_MUSIC`  
- **<SUFIXO>**: Indica a etapa do pipeline

### Mapeamento de Sufixos

| Sufixo | Sigla | Significado                          | Exemplos de uso                                      |
|--------|-------|--------------------------------------|------------------------------------------------------|
| FNG    | FNG   | Features Engineering / ABT           | Features, base de treino, enriquecimento             |
| TRN    | TRN   | Treinamento / Target                 | Target, dataset final de treino, activated/canceled  |
| INF    | INF   | Inferências / Predições / Score      | Predictions, inference, output de modelo             |
| ETL    | ETL   | Processamento / Raw / Histórico      | Raw data, base preprocessada, enriquecida            |
| SND    | SND   | Envio de alertas / Mailing / Semaforo| Mailing alto risco, semáforo churn, envios finais    |

### Exemplos de Tasks Padronizadas

- `TK_DS_CHURN_INADIMPLENCIA_CNPJ_INF`  
- `TK_DS_MAU_PAGADOR_CPF_FNG`  
- `TK_DS_VISAO_CLIENTE_B2B_ETL`  
- `TK_DS_SEMAFORO_CHURN_CORP_SND`  
- `TK_DS_PROPENSAO_NOMO_MUSIC_INF`

## 2. Proposta de Padronização para Tabelas

Objetivo: Alinhar nomes de tabelas com o padrão das tasks, facilitando a relação task → tabela.

Sugestão de formato para tabelas:  
**`[DS_ ou vazio]_<PROJETO>_<SUFIXO>`**  
(ou manter `TK_DS_` se quiser forte ligação com tasks)

### Sufixos Recomendados para Tabelas

| Sufixo tabela | Equivalente task | Quando usar na tabela                              |
|---------------|------------------|----------------------------------------------------|
| _FNG          | FNG              | Features, ABT, base de treino, enriquecimento      |
| _TRN          | TRN              | Target, dataset final de treino                    |
| _INF          | INF              | Predições, scores, inferências, output modelo      |
| _ETL          | ETL              | Raw, preprocess, histórico, base intermediária     |
| _SND          | SND              | Mailing, alertas, semáforo final para consumo      |
| _AUX / _TMP   | —                | Tabelas auxiliares, temporárias, logs, dev         |

### Exemplos de Tabelas Antigas → Sugestão Nova

| #  | Tabela Atual                              | Projeto Sugerido               | Sufixo Sugerido | Nome Sugerido (exemplo)                        | Comentário                                      |
|----|-------------------------------------------|--------------------------------|-----------------|------------------------------------------------|-------------------------------------------------|
| 3  | BASE_ABT_CHURNMPE                         | CHURN_MPE                      | FNG             | DS_CHURN_MPE_FNG                               | ABT base features churn MPE                     |
| 6  | BASE_ABT_INFERENCE_CHURNMPE               | CHURN_MPE                      | INF             | DS_CHURN_MPE_INF_BASE                          | Base para inferência                            |
| 27 | CHURN_BL_MPE_FEATURE_IMPORTANCE           | CHURN_BL_MPE                   | FNG             | DS_CHURN_BL_MPE_FNG_IMPORTANCE                 | Features + importância                          |
| 29 | CHURN_BL_MPE_INFERENCE                    | CHURN_BL_MPE                   | INF             | DS_CHURN_BL_MPE_INF                            | Inferências                                     |
| 31 | CHURN_CORP_PREDICTIONS                    | CHURN_CORP                     | INF             | DS_CHURN_CORP_INF                              | Predições corp                                  |
| 36 | CHURN_INADIMPLENCIA_CNPJ_PREDICTIONS      | CHURN_INADIMPLENCIA_CNPJ       | INF             | DS_CHURN_INADIMPLENCIA_CNPJ_INF                | Predições inadimplência CNPJ                    |
| 37 | CHURN_INADIMPLENCIA_CNPJ_TARGET           | CHURN_INADIMPLENCIA_CNPJ       | TRN             | DS_CHURN_INADIMPLENCIA_CNPJ_TRN                | Target inadimplência CNPJ                       |
| 87 | MAUPAGADOR_CNPJ_PREDICTIONS               | MAU_PAGADOR_CNPJ               | INF             | DS_MAU_PAGADOR_CNPJ_INF                        | Predições mau pagador CNPJ                      |
| 96 | PREDICOES_PROPENSAO_COMPRA_NOMO_MUSIC     | PROPENSAO_NOMO_MUSIC           | INF             | DS_PROPENSAO_NOMO_MUSIC_INF                    | Predições propensão Nomo Music                  |
| 99 | SEMAFORO_CHURN_CORP_RISK                  | SEMAFORO_CHURN_CORP            | SND             | DS_SEMAFORO_CHURN_CORP_SND                     | Semáforo risco churn corp                       |
| 118| VISAO_CLIENTE_B2B                         | VISAO_CLIENTE_B2B              | ETL             | DS_VISAO_CLIENTE_B2B_ETL                       | Visão cliente B2B                               |
| 21 | BEEGOL_DAILY_2025                         | BEEGOL                         | ETL             | DS_BEEGOL_DAILY_ETL                            | Beegol daily processado                         |
| 24 | BEEGOL_MAILING_EMP_RES                    | BEEGOL                         | SND             | DS_BEEGOL_MAILING_SND                          | Mailing Beegol                                  |

## 3. Recomendações de Implementação

1. **Decisão de prefixo para tabelas**  
   - Opção A: `DS_<PROJETO>_<SUFIXO>` (recomendado – simples e claro)  
   - Opção B: `TK_DS_<PROJETO>_<SUFIXO>` (forte ligação com tasks)  
   - Opção C: Manter nomes atuais + adicionar sufixo apenas (ex: `CHURN_CORP_PREDICTIONS_INF`)

2. **Passos sugeridos**  
   - Criar tabela de mapeamento no Snowflake: `META_PADRONIZACAO_NOMES`  
     colunas: nome_atual, projeto, sufixo_sugerido, nome_novo, status (Pendente/Feito)  
   - Script de rename (exemplo):  
     ```sql
     ALTER TABLE DWDEV.DATASCIENCE.CHURN_CORP_PREDICTIONS 
     RENAME TO DWDEV.DATASCIENCE.DS_CHURN_CORP_INF;