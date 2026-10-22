# Ficha do dataset — Respiratory Symptoms and Treatment

> Números calculados pelo notebook de auditoria (`Base_Respiratoria`), com tabelas em `outputs/`.

## 1. Origem

| Item | Descrição |
|---|---|
| Nome | Respiratory Symptoms and Treatment |
| Plataforma | Kaggle — https://www.kaggle.com/datasets/abbotpatcher/respiratory-symptoms-and-treatment |
| Autor / publicador | American Health Info (usuário `abbotpatcher`) |
| Publicação / versão | 16/04/2021 · Versão 5 - arquivo `respiratory symptoms and treatment.csv` (2,3 MB) |
| Fonte declarada | Departamento de saúde pública (Albuquerque / Novo México), segundo a descrição do autor. **Não verificável** a partir do arquivo |
| Real ou sintético | O autor **não declara** se os dados são reais ou sintéticos. Os indícios internos (seção 5) apontam para geração ou replicação em lotes |
| Instituição responsável / ética | Não identificadas; sem menção a aprovação ética (CEP) |
| Anotação | Associação de sintoma, idade, sexo e gravidade com um diagnóstico e um tratamento, em texto livre e sem códigos padrão (CID-10 e SNOMED CT foram atribuídos pela auditoria) |
| Citação | American Health Info (2021). *Respiratory Symptoms and Treatment* (v5) [conjunto de dados]. Kaggle. URL acima |
| Integridade | SHA-256 do CSV: `a88edab166baa44e974c8a00e4cc4965867c923a7c8871afe33f40a9e6e39226` |

## 2. Licença

- **Declarada pelo autor**: Open Database License (ODbL) para a base e Database Contents License (DbCL) para o conteúdo.
- **Na prática**: permite uso e redistribuição **com atribuição (autor e URL) e compartilhamento sob a mesma licença**. A licença não atesta qualidade nem validade clínica: o uso clínico direto está **bloqueado** pela falta de proveniência verificável (seção 5). Uso restrito a ambiente de *sandbox* técnico.

## 3. Tamanho

| Medida | Valor |
|---|---|
| Linhas × colunas | 38.537 × 6 |
| Linhas únicas (cópia exata) | 903 (901 após normalizar texto) |
| Duplicidade exata | **97,7 %** das linhas (37.634 cópias) |
| Tamanho efetivo por doença | 15 a 125 linhas únicas por doença (VSR: 15; pneumonia: 125), contra 504 a 6.144 linhas brutas |

## 4. Variáveis

| Variável | Tipo / valores | Nulos |
|---|---|---|
| `Symptoms` | Texto livre em inglês; 92 variantes → 77 após normalizar caixa e espaços | 1,8 % |
| `Age` | Numérica, 1 a 98 anos (mediana 30) | 0,9 % |
| `Sex` | male (55,2 %), female (40,0 %), not to say (2,5 %) | 2,4 % |
| `Disease` | 18 diagnósticos em texto. Inclui `chronic cough`, que é um sintoma | 0,9 % |
| `Treatment` | 33 variantes de texto (com erros como `oxyzen` e `inhealer`). Inclui `x-ray` e `consult a doctor`, que não são tratamentos | 7,4 % |
| `Nature` | high (58,7 %), medium (34,4 %), low (1,1 %); interpretada como gravidade na auditoria | 5,7 % |

**Não existem**: identificador de paciente, data, unidade de saúde nem códigos padrão.

**Terminologia (feita na auditoria)**: mapeamento para CID-10 e SNOMED CT com termos em português; cobre 99,1 % das linhas quanto à doença e 97,0 % quanto ao sintoma (5 variantes de texto, cerca de 3 % das linhas, ficam sem mapa).

## 5. Vieses e achados da auditoria

1. **Duplicação massiva**: 97,7 % das linhas são cópias exatas. 97 % dos perfis se repetem em múltiplos de 24 e cerca de 90 % das reaparições ocorrem a distâncias fixas no arquivo (949, 644 e 1.593 linhas), compatível com replicação em lotes. É um **indício**, não uma prova.
2. **Inconsistências clínicas**: 21,5 % das linhas disparam ao menos um alerta de plausibilidade (limite inferior, com apenas 3 regras): `chronic cough` como doença, bronquite sempre tratada com antibiótico e pneumotórax só com soro fisiológico isotônico.
3. **Proveniência incompleta**: cumpre **6 de 12** critérios mínimos. Autor, licença e metodologia estão declarados, mas faltam rastreabilidade (sem identificadores), declaração de natureza real ou sintética, aprovação ética e validação clínica.
4. **Variável-alvo contaminada**: cerca de 15 % das linhas não trazem intervenção terapêutica válida (nulos, `x-ray` e `consult a doctor`).
5. **Perfil demográfico**: 29 % têm até 12 anos e 28,5 % têm 60 anos ou mais; 55 % são do sexo masculino. A ausência do sexo concentra-se em algumas doenças (21 % em `chronic cough`, 16 % na síndrome do desconforto respiratório agudo), isto é, a falta de dado não é aleatória.
6. **Classes desbalanceadas**: após desduplicar, algumas doenças ficam com poucas dezenas de perfis.

## 6. Riscos de vazamento e desempenho

| Risco | Evidência | Mitigação |
|---|---|---|
| Cópias idênticas em treino e teste | Com partição aleatória sobre linhas brutas, o recomendador de tratamentos acerta **93 %** (Top-1) | Desduplicar e particionar por **grupo de perfil** |
| Perfis quase idênticos (só a idade exata difere) | Em uma partição aleatória de perfis únicos, cerca de metade do teste compartilha perfil com o treino | Partição por perfil (doença, sintoma, sexo, gravidade e faixa etária) |
| Queda ao remover o vazamento | Sem vazamento, o Top-1 cai para **77 %** (média de 20 repetições; Top-3 fica em 98 %). Usando só a doença, o Top-1 é **68 %** | Reportar sempre a métrica sem vazamento, com desvio-padrão |
| Tratamento quase determinado pela doença | 14 de 17 doenças têm no máximo 3 tratamentos distintos: o Top-3 é quase trivial | Usar o Top-1 como medida principal |
| Ausência de metadados | Sem paciente, data ou unidade: não é possível particionar por paciente nem por tempo | Documentar como limitação |

## 7. Recomendações de uso

- **Proibido**: treinar, validar ou exibir recomendações com respaldo clínico; calcular indicadores epidemiológicos oficiais; misturar com dados reais de pacientes.
- **Permitido**: *sandbox* técnico para testar ingestão em camada bronze, mapeamento CID-10/SNOMED, dicionário de sintomas em português e código de avaliação de recomendadores.
- **Diretriz de governança proposta**: nenhuma base de dados abertos entra na camada bronze sem ficha de proveniência aprovada e checagem de duplicadas abaixo de 5 %.
