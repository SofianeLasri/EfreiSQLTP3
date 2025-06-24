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

## 1.4 Test des différentes opérations

### 1. Égalité exacte

Requête :

```sql
EXPLAIN ANALYZE
SELECT tconst, primary_title
FROM title_basics
WHERE primary_title = 'The Godfather';
```

Résultat :

```
"QUERY PLAN"
"Gather  (cost=1000.00..236197.05 rows=133 width=31) (actual time=11.316..428.288 rows=43 loops=1)"
"  Workers Planned: 2"
"  Workers Launched: 2"
"  ->  Parallel Seq Scan on title_basics  (cost=0.00..235183.75 rows=55 width=31) (actual time=10.557..399.572 rows=14 loops=3)"
"        Filter: ((primary_title)::text = 'The Godfather'::text)"
"        Rows Removed by Filter: 3911606"
"Planning Time: 0.090 ms"
"JIT:"
"  Functions: 12"
"  Options: Inlining false, Optimization false, Expressions true, Deforming true"
"  Timing: Generation 1.522 ms (Deform 0.570 ms), Inlining 0.000 ms, Optimization 0.965 ms, Emission 13.085 ms, Total 15.572 ms"
"Execution Time: 428.836 ms"
```

Utilise l'index.

### 2. Préfixe

Requête :

```sql
EXPLAIN ANALYZE
SELECT tconst, primary_title
FROM title_basics
WHERE primary_title LIKE 'The%'
LIMIT 100;
```

Résultat :

```
"QUERY PLAN"
"Limit  (cost=0.00..50.74 rows=100 width=31) (actual time=0.021..0.120 rows=100 loops=1)"
"  ->  Seq Scan on title_basics  (cost=0.00..320773.80 rows=632196 width=31) (actual time=0.020..0.112 rows=100 loops=1)"
"        Filter: ((primary_title)::text ~~ 'The%'::text)"
"        Rows Removed by Filter: 543"
"Planning Time: 0.100 ms"
"Execution Time: 0.161 ms"
```

Utilise l'index.

### 3. Suffixe

Requête :

```sql
EXPLAIN ANALYZE
SELECT tconst, primary_title
FROM title_basics
WHERE primary_title LIKE '%Father'
LIMIT 100;
```

Résultat :

```
"QUERY PLAN"
"Limit  (cost=1000.00..23558.78 rows=100 width=31) (actual time=0.506..37.355 rows=100 loops=1)"
"  ->  Gather  (cost=1000.00..236288.05 rows=1043 width=31) (actual time=0.505..37.337 rows=100 loops=1)"
"        Workers Planned: 2"
"        Workers Launched: 2"
"        ->  Parallel Seq Scan on title_basics  (cost=0.00..235183.75 rows=435 width=31) (actual time=0.300..10.758 rows=35 loops=3)"
"              Filter: ((primary_title)::text ~~ '%Father'::text)"
"              Rows Removed by Filter: 76341"
"Planning Time: 0.107 ms"
"Execution Time: 37.437 ms"
```

N'utilise pas l'index

### 4. Sous-chaîne

Requête :

```sql
EXPLAIN ANALYZE
SELECT tconst, primary_title
FROM title_basics
WHERE primary_title LIKE '%God%'
LIMIT 100;
```

Résultat :

```
"QUERY PLAN"
"Limit  (cost=1000.00..23558.78 rows=100 width=31) (actual time=0.694..31.725 rows=100 loops=1)"
"  ->  Gather  (cost=1000.00..236288.05 rows=1043 width=31) (actual time=0.692..31.709 rows=100 loops=1)"
"        Workers Planned: 2"
"        Workers Launched: 2"
"        ->  Parallel Seq Scan on title_basics  (cost=0.00..235183.75 rows=435 width=31) (actual time=0.137..3.403 rows=34 loops=3)"
"              Filter: ((primary_title)::text ~~ '%God%'::text)"
"              Rows Removed by Filter: 20158"
"Planning Time: 0.085 ms"
"Execution Time: 31.798 ms"
```

N'utilise pas l'index

### 5. Ordre

Requête :

```sql
EXPLAIN ANALYZE
SELECT tconst, primary_title
FROM title_basics
ORDER BY primary_title
LIMIT 100;
```

Résultat :

```
"QUERY PLAN"
"Limit  (cost=410881.78..410893.44 rows=100 width=31) (actual time=1314.994..1324.468 rows=100 loops=1)"
"  ->  Gather Merge  (cost=410881.78..1552162.12 rows=9781720 width=31) (actual time=1312.654..1322.120 rows=100 loops=1)"
"        Workers Planned: 2"
"        Workers Launched: 2"
"        ->  Sort  (cost=409881.75..422108.90 rows=4890860 width=31) (actual time=1293.514..1293.518 rows=77 loops=3)"
"              Sort Key: primary_title"
"              Sort Method: top-N heapsort  Memory: 34kB"
"              Worker 0:  Sort Method: top-N heapsort  Memory: 32kB"
"              Worker 1:  Sort Method: top-N heapsort  Memory: 33kB"
"              ->  Parallel Seq Scan on title_basics  (cost=0.00..222956.60 rows=4890860 width=31) (actual time=2.162..514.664 rows=3911620 loops=3)"
"Planning Time: 0.081 ms"
"JIT:"
"  Functions: 7"
"  Options: Inlining false, Optimization false, Expressions true, Deforming true"
"  Timing: Generation 0.500 ms (Deform 0.287 ms), Inlining 0.000 ms, Optimization 0.758 ms, Emission 7.955 ms, Total 9.213 ms"
"Execution Time: 1324.838 ms"
```

Utilise l'index.

