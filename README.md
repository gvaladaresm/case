%sql

-- Base com flags treated e post, garantindo tipos
CREATE OR REPLACE TEMP VIEW did_base AS
SELECT
  mes,
  cliente_id,
  CAST(tpv_pix AS DOUBLE) AS tpv_pix,
  CASE WHEN grupo_experimento = 'tratamento' THEN 1 ELSE 0 END AS treated,
  CASE WHEN mes >= DATE '2024-06-01' THEN 1 ELSE 0 END AS post
FROM vw_base_mensal_grupos_tratamento
WHERE mes IS NOT NULL;

-- (Opcional) Deduplica por cliente-mês e remove NAs
CREATE OR REPLACE TEMP VIEW did_base_clean AS
SELECT
  mes, cliente_id,
  MAX(treated) AS treated,
  MAX(post)    AS post,
  MAX(tpv_pix) AS tpv_pix
FROM did_base
WHERE tpv_pix IS NOT NULL
GROUP BY mes, cliente_id;

-- Médias por grupo e período
WITH grp AS (
  SELECT treated, post, AVG(tpv_pix) AS avg_tpv
  FROM did_base_clean
  GROUP BY treated, post
),
-- Deltas (pós - pré) dentro de cada grupo
deltas AS (
  SELECT
    treated,
    MAX(CASE WHEN post = 1 THEN avg_tpv END)
    - MAX(CASE WHEN post = 0 THEN avg_tpv END) AS delta
  FROM grp
  GROUP BY treated
)
-- Estimador DiD: (Δ Trat) - (Δ Ctrl)
SELECT
  MAX(CASE WHEN treated = 1 THEN delta END)
  - MAX(CASE WHEN treated = 0 THEN delta END) AS did_tpv_pix
FROM deltas;
