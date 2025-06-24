# Exercice 1

## 1.1 Analyse sans index

Requête :

```sql
EXPLAIN ANALYZE
SELECT tconst, primary_title, title_type, start_year
FROM title_basics
WHERE primary_title LIKE 'The%';
```

Résultat :

```
"QUERY PLAN"
"Gather  (cost=1000.00..180015.60 rows=4351 width=620) (actual time=4.314..1139.873 rows=605129 loops=1)"
"  Workers Planned: 2"
"  Workers Launched: 2"
"  ->  Parallel Seq Scan on title_basics  (cost=0.00..178580.50 rows=1813 width=620) (actual time=5.442..1083.161 rows=201710 loops=3)"
"        Filter: ((primary_title)::text ~~ 'The%'::text)"
"        Rows Removed by Filter: 3709910"
"Planning Time: 0.205 ms"
"JIT:"
"  Functions: 12"
"  Options: Inlining false, Optimization false, Expressions true, Deforming true"
"  Timing: Generation 1.891 ms (Deform 0.959 ms), Inlining 0.000 ms, Optimization 1.036 ms, Emission 15.126 ms, Total 18.053 ms"
"Execution Time: 1162.875 ms"
```

## 1.2 Création d'un index B-Tree

Requête :

```sql
CREATE INDEX idx_title_basics_primary_title ON title_basics (primary_title);
```

## 1.3 Analyse après indexation

Résultat :

```
"QUERY PLAN"
"Gather  (cost=1000.00..299403.35 rows=632196 width=44) (actual time=5.662..506.038 rows=605129 loops=1)"
"  Workers Planned: 2"
"  Workers Launched: 2"
"  ->  Parallel Seq Scan on title_basics  (cost=0.00..235183.75 rows=263415 width=44) (actual time=4.787..454.861 rows=201710 loops=3)"
"        Filter: ((primary_title)::text ~~ 'The%'::text)"
"        Rows Removed by Filter: 3709910"
"Planning Time: 0.211 ms"
"JIT:"
"  Functions: 12"
"  Options: Inlining false, Optimization false, Expressions true, Deforming true"
"  Timing: Generation 1.318 ms (Deform 0.616 ms), Inlining 0.000 ms, Optimization 0.872 ms, Emission 13.263 ms, Total 15.453 ms"
"Execution Time: 527.700 ms"
```