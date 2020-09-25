# quoting for rendering ordered grouped table

    Code
      sql_render(out)
    Output
      <SQL> SELECT *
      FROM `test-verb-arrange`
      ORDER BY `y`

# arrange renders correctly (#373)

    Code
      # # arrange renders correctly
    Code
      lf <- lazy_frame(a = 1:3, b = 3:1)
    Code
      # basic
    Code
      lf %>% arrange(a)
    Output
      <SQL>
      SELECT *
      FROM `df`
      ORDER BY `a`
    Code
      # double arrange
    Code
      lf %>% arrange(a) %>% arrange(b)
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT *
      FROM `df`
      ORDER BY `b`
    Code
      # remove ordered by
    Code
      lf %>% arrange(a) %>% select(-a)
    Output
      <SQL>
      SELECT `b`
      FROM `df`
      ORDER BY `a`
    Code
      lf %>% arrange(a) %>% select(-a) %>% arrange(b)
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT `b`
      FROM `df`
      ORDER BY `b`
    Code
      # un-arrange
    Code
      lf %>% arrange(a) %>% arrange()
    Output
      <SQL>
      SELECT *
      FROM `df`
      ORDER BY `a`
    Code
      lf %>% arrange(a) %>% select(-a) %>% arrange()
    Output
      <SQL>
      SELECT `b`
      FROM `df`
      ORDER BY `a`
    Code
      # use order
    Code
      lf %>% arrange(a) %>% select(-a) %>% mutate(c = lag(b))
    Output
      <SQL>
      SELECT `b`, LAG(`b`, 1, NULL) OVER (ORDER BY `a`) AS `c`
      FROM `df`
      ORDER BY `a`

# arrange renders correctly for single-table verbs (#373)

    Code
      lf <- lazy_frame(a = 1:3, b = 3:1)
    Code
      # head
    Code
      lf %>% head(1) %>% arrange(a)
    Output
      <SQL>
      SELECT *
      FROM (SELECT *
      FROM `df`
      LIMIT 1) `q01`
      ORDER BY `a`
    Code
      lf %>% arrange(a) %>% head(1)
    Output
      <SQL>
      SELECT *
      FROM `df`
      ORDER BY `a`
      LIMIT 1
    Code
      lf %>% arrange(a) %>% head(1) %>% arrange(b)
    Output
      <SQL>
      SELECT *
      FROM (SELECT *
      FROM `df`
      ORDER BY `a`
      LIMIT 1) `q01`
      ORDER BY `b`
    Code
      # mutate
    Code
      lf %>% mutate(a = b) %>% arrange(a)
    Output
      <SQL>
      SELECT `b` AS `a`, `b`
      FROM `df`
      ORDER BY `a`
    Code
      # complex mutate
    Code
      lf %>% arrange(a) %>% mutate(a = b) %>% arrange(a)
    Output
      <SQL>
      SELECT `b` AS `a`, `b`
      FROM `df`
      ORDER BY `a`
    Code
      lf %>% arrange(a) %>% mutate(a = 1) %>% arrange(b)
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT 1.0 AS `a`, `b`
      FROM `df`
      ORDER BY `b`
    Code
      lf %>% arrange(a) %>% mutate(b = a) %>% arrange(b)
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT `a`, `a` AS `b`
      FROM `df`
      ORDER BY `b`
    Code
      lf %>% arrange(a) %>% mutate(b = 1) %>% arrange(b)
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT `a`, 1.0 AS `b`
      FROM `df`
      ORDER BY `b`
    Code
      lf %>% mutate(a = -a) %>% arrange(a) %>% mutate(a = -a)
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT -`a` AS `a`, `b`
      FROM (SELECT -`a` AS `a`, `b`
      FROM `df`) `q01`

# arrange renders correctly for joins (#373)

    Code
      lf <- lazy_frame(a = 1:3, b = 3:1)
    Code
      rf <- lazy_frame(a = 1:3, c = 4:6)
    Code
      # join
    Code
      lf %>% arrange(a) %>% left_join(rf)
    Message <message>
      Joining, by = "a"
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT `LHS`.`a` AS `a`, `LHS`.`b` AS `b`, `RHS`.`c` AS `c`
      FROM (SELECT *
      FROM `df`) `LHS`
      LEFT JOIN `df` AS `RHS`
      ON (`LHS`.`a` = `RHS`.`a`)
      
    Code
      lf %>% arrange(b) %>% left_join(rf)
    Message <message>
      Joining, by = "a"
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT `LHS`.`a` AS `a`, `LHS`.`b` AS `b`, `RHS`.`c` AS `c`
      FROM (SELECT *
      FROM `df`) `LHS`
      LEFT JOIN `df` AS `RHS`
      ON (`LHS`.`a` = `RHS`.`a`)
      
    Code
      lf %>% left_join(rf) %>% arrange(a)
    Message <message>
      Joining, by = "a"
    Output
      <SQL>
      SELECT *
      FROM (SELECT `LHS`.`a` AS `a`, `LHS`.`b` AS `b`, `RHS`.`c` AS `c`
      FROM `df` AS `LHS`
      LEFT JOIN `df` AS `RHS`
      ON (`LHS`.`a` = `RHS`.`a`)
      ) `q01`
      ORDER BY `a`
    Code
      lf %>% left_join(rf) %>% arrange(b)
    Message <message>
      Joining, by = "a"
    Output
      <SQL>
      SELECT *
      FROM (SELECT `LHS`.`a` AS `a`, `LHS`.`b` AS `b`, `RHS`.`c` AS `c`
      FROM `df` AS `LHS`
      LEFT JOIN `df` AS `RHS`
      ON (`LHS`.`a` = `RHS`.`a`)
      ) `q01`
      ORDER BY `b`
    Code
      lf %>% left_join(rf %>% arrange(a))
    Message <message>
      Joining, by = "a"
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT `LHS`.`a` AS `a`, `LHS`.`b` AS `b`, `RHS`.`c` AS `c`
      FROM `df` AS `LHS`
      LEFT JOIN (SELECT *
      FROM `df`) `RHS`
      ON (`LHS`.`a` = `RHS`.`a`)
      
    Code
      lf %>% left_join(rf %>% arrange(c))
    Message <message>
      Joining, by = "a"
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT `LHS`.`a` AS `a`, `LHS`.`b` AS `b`, `RHS`.`c` AS `c`
      FROM `df` AS `LHS`
      LEFT JOIN (SELECT *
      FROM `df`) `RHS`
      ON (`LHS`.`a` = `RHS`.`a`)
      

# arrange renders correctly for semi-joins (#373)

    Code
      lf <- lazy_frame(a = 1:3, b = 3:1)
    Code
      rf <- lazy_frame(a = 1:3, c = 4:6)
    Code
      # semi_join
    Code
      lf %>% arrange(a) %>% semi_join(rf)
    Message <message>
      Joining, by = "a"
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT * FROM (SELECT *
      FROM `df`) `LHS`
      WHERE EXISTS (
        SELECT 1 FROM `df` AS `RHS`
        WHERE (`LHS`.`a` = `RHS`.`a`)
      )
    Code
      lf %>% arrange(b) %>% semi_join(rf)
    Message <message>
      Joining, by = "a"
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT * FROM (SELECT *
      FROM `df`) `LHS`
      WHERE EXISTS (
        SELECT 1 FROM `df` AS `RHS`
        WHERE (`LHS`.`a` = `RHS`.`a`)
      )
    Code
      lf %>% semi_join(rf) %>% arrange(a)
    Message <message>
      Joining, by = "a"
    Output
      <SQL>
      SELECT *
      FROM (SELECT * FROM `df` AS `LHS`
      WHERE EXISTS (
        SELECT 1 FROM `df` AS `RHS`
        WHERE (`LHS`.`a` = `RHS`.`a`)
      )) `q01`
      ORDER BY `a`
    Code
      lf %>% semi_join(rf) %>% arrange(b)
    Message <message>
      Joining, by = "a"
    Output
      <SQL>
      SELECT *
      FROM (SELECT * FROM `df` AS `LHS`
      WHERE EXISTS (
        SELECT 1 FROM `df` AS `RHS`
        WHERE (`LHS`.`a` = `RHS`.`a`)
      )) `q01`
      ORDER BY `b`
    Code
      lf %>% semi_join(rf %>% arrange(a))
    Message <message>
      Joining, by = "a"
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT * FROM `df` AS `LHS`
      WHERE EXISTS (
        SELECT 1 FROM (SELECT *
      FROM `df`) `RHS`
        WHERE (`LHS`.`a` = `RHS`.`a`)
      )
    Code
      lf %>% semi_join(rf %>% arrange(c))
    Message <message>
      Joining, by = "a"
    Warning <warning>
      ORDER BY is ignored in subqueries without LIMIT
      i Do you need to move arrange() later in the pipeline or use window_order() instead?
    Output
      <SQL>
      SELECT * FROM `df` AS `LHS`
      WHERE EXISTS (
        SELECT 1 FROM (SELECT *
      FROM `df`) `RHS`
        WHERE (`LHS`.`a` = `RHS`.`a`)
      )

# arrange renders correctly for set operations (#373)

    Code
      lf <- lazy_frame(a = 1:3, b = 3:1)
    Code
      rf <- lazy_frame(a = 1:3, c = 4:6)
    Code
      # setop
    Code
      lf %>% union_all(rf) %>% arrange(a)
    Output
      <SQL>
      SELECT *
      FROM ((SELECT `a`, `b`, NULL AS `c`
      FROM `df`)
      UNION ALL
      (SELECT `a`, NULL AS `b`, `c`
      FROM `df`)) `q01`
      ORDER BY `a`
    Code
      lf %>% arrange(a) %>% union_all(rf)
    Output
      <SQL>
      (SELECT `a`, `b`, NULL AS `c`
      FROM `df`
      ORDER BY `a`)
      UNION ALL
      (SELECT `a`, NULL AS `b`, `c`
      FROM `df`)
    Code
      lf %>% union_all(rf %>% arrange(a))
    Output
      <SQL>
      (SELECT `a`, `b`, NULL AS `c`
      FROM `df`)
      UNION ALL
      (SELECT `a`, NULL AS `b`, `c`
      FROM `df`
      ORDER BY `a`)
