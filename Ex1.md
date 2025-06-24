1.1 Analyse sans index

    EXPLAIN ANALYZE
    select primary_title
    from title_basics
    Where primary_title LIKE 'The%';

    "Gather  (cost=1000.00..299352.76 rows=633777 width=20) (actual time=19.334..906.014 rows=605129 loops=1)"
    "  Workers Planned: 2"
    "  Workers Launched: 2"
    "  ->  Parallel Seq Scan on title_basics  (cost=0.00..234975.06 rows=264074 width=20) (actual time=10.390..845.296 rows=201710 loops=3)"
    "        Filter: ((primary_title)::text ~~ 'The%'::text)"
    "        Rows Removed by Filter: 3709910"
    "Planning Time: 0.693 ms"
    "JIT:"
    "  Functions: 12"
    "  Options: Inlining false, Optimization false, Expressions true, Deforming true"
    "  Timing: Generation 2.065 ms (Deform 0.640 ms), Inlining 0.000 ms, Optimization 2.660 ms, Emission 28.316 ms, Total 33.041 ms"
    "Execution Time: 944.671 ms"

1.2 Création d'un index B-tree

    create index IDX_primary_title ON title_basics(primary_title);

1.3 Analyse après indexation

    "Gather  (cost=1000.00..299352.76 rows=633777 width=20) (actual time=76.009..1562.525 rows=605129 loops=1)"
    "  Workers Planned: 2"
    "  Workers Launched: 2"
    "  ->  Parallel Seq Scan on title_basics  (cost=0.00..234975.06 rows=264074 width=20) (actual time=29.736..1424.937 rows=201710 loops=3)"
    "        Filter: ((primary_title)::text ~~ 'The%'::text)"
    "        Rows Removed by Filter: 3709910"
    "Planning Time: 1.989 ms"
    "JIT:"
    "  Functions: 12"
    "  Options: Inlining false, Optimization false, Expressions true, Deforming true"
    "  Timing: Generation 12.419 ms (Deform 3.318 ms), Inlining 0.000 ms, Optimization 16.210 ms, Emission 72.125 ms, Total 100.754 ms"
    "Execution Time: 1610.846 ms"
    
    L'index rend la requête plus lente

1.4 Test des différentes opérations

    Oui
    Non
    Non
    Non
    Oui

1.5 Analyse et réflexion

    1) pour les opérations d'égalité exacte et les comparaisons ordonnées
    
    2) PostgreSQL utilise un optimiseur de coût qui compare le coût estimé d'utiliser l'index avec un scan séquentiel. Dans LIKE 'The%', l'index n'est pas utilisé car la requête retourne 605 129 lignes sur environ 11 millions, soit 5% de la table. 
    L'optimiseur considère que pour accéder à autant de lignes dispersées, il est plus rapide de lire séquentiellement toute la table que d’utiliser les accès de l’index. 
    
    3) Pour les recherches très sélectives qui retournent moins de 1-5% des lignes de la table
