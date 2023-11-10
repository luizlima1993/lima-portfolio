# Evaluating Equipment Performance through Pareto Analysis using SQL

## The most relevant root causes affecting the equipment performance were identified by analyzing three different databases and performing some feature engineering to allow an accurate ranking the 20% problems that account for 80% of performance losses.

### The databases used were:

* FactProdPerf
* PerfMensal
* FactDemoras

### The code is illustrated below:

```sql
-- Neste exemplo tive que complicar para praticar com datas e cast,deixei tudo explicito
WITH PerfMensal AS
(
	SELECT 
		P.COD_DIA,
		P.COD_MES,
		MONTH(CAST (P.COD_DIA AS Date)) AS MES,
		YEAR(CAST (P.COD_DIA AS Date)) AS ANO,
		P.COD_SUBLINEA,
		(24 * P.UTDISPONIBLESTD * P.UTNETASTD)/10000 AS TnetoProg,
		P.PRODORIGINAL * 24 AS ProdProg
	FROM FactProdPerf AS P
	WHERE P.COD_SUBLINEA = 'B72'
),
PnetaMensal AS
(
	SELECT
		PM.COD_MES,
		SUM(PM.ProdProg)/SUM(PM.TnetoProg) AS PnetaProg
	FROM PerfMensal PM
	GROUP BY PM.COD_MES
),
PerfDem AS
(
	SELECT 
		D.COD_MES,
		D.DES_SUBCONCEPTO,
		D.HRSREALMENSUAL,
		PM.PnetaProg
	FROM FactDemoras D
		LEFT JOIN PnetaMensal PM ON (D.COD_MES = PM.COD_MES)
	WHERE D.COD_SUBLINEA = 'B72'
)


SELECT
	PD.COD_MES AS 'YEAR_MONTH',
	PD.DES_SUBCONCEPTO AS 'ROOT CAUSE',
	PD.HRSREALMENSUAL AS 'HOURS',
	SUM(PD.HRSREALMENSUAL) OVER (PARTITION BY PD.COD_MES ORDER BY PD.HRSREALMENSUAL DESC) AS RollSum,
	PD.PnetaProg AS 'Productivity'
FROM PerfDem PD
WHERE PD.HRSREALMENSUAL > 0

SELECT * FROM PnetaMensal
;
```

### The final result showed in the table below allows us to get to a conclusion:

| Product | Day  | Year | Type       |
| :------ | ---- | ---- | ---------- |
| PC      | 1    | 2021 | electronic |
| Shampoo | 2    | 2022 | healthcare |
| PC      | 3    | 2023 | electronic |
| Water   | 4    | 2024 | utility    |

Thanks for your time.