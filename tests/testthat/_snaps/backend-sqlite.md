# custom aggregates translated

    Code
      (expect_error(translate_sql(quantile(x, 0.5, na.rm = TRUE), window = FALSE)))
    Output
      <error/rlang_error>
      Error in `quantile()`:
      ! `quantile()` is not available in this SQL variant.
    Code
      (expect_error(translate_sql(quantile(x, 0.5, na.rm = TRUE), window = TRUE)))
    Output
      <error/rlang_error>
      Error in `quantile()`:
      ! `quantile()` is not available in this SQL variant.

# custom SQL translation

    Code
      left_join(lf, lf, by = "x", na_matches = "na")
    Output
      <SQL>
      SELECT `df_LHS`.`x` AS `x`
      FROM `df` AS `df_LHS`
      LEFT JOIN `df` AS `df_RHS`
        ON (`df_LHS`.`x` IS `df_RHS`.`x`)

# case_when translates correctly to ELSE when TRUE ~ is used

    Code
      translate_sql(case_when(x == 1L ~ "yes", x == 0L ~ "no", TRUE ~ "undefined"),
      con = simulate_sqlite())
    Output
      <SQL> CASE WHEN (`x` = 1) THEN 'yes' WHEN (`x` = 0) THEN 'no' ELSE 'undefined' END

# can explain a query

    Code
      db %>% filter(x > 2) %>% explain()
    Output
      <SQL>
      SELECT *
      FROM `test`
      WHERE (`x` > 2.0)
      
      <PLAN>
        id parent notused                                        detail
      1  2      0       0 SEARCH test USING COVERING INDEX test_x (x>?)

