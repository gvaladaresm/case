%sql
CREATE OR REPLACE TEMP VIEW vw_base_mensal_grupos_tratamento AS
SELECT
  mes,
  cliente_id,
  segmento,
  uf,
  tempo_cliente_meses,
  tpv_total_medio,
  saldo_medio_conta,
  tpv_pix,
  receita_pix,
  saldo_conta,
  grupo_experimento,
  usuario_pix,
  participou_promocao,
  aceitou_pgto_pix
FROM vw_base_mensal_enriquecida
WHERE grupo_experimento IS NOT NULL
