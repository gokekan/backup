오라클/티베로 resultMap VO 추출 쿼리
===

```sql
SELECT COLUMN_NAME || ',' AS C1
  , LOWER (COLUMN_NAME) || ',' AS C2
  , LOWER (SUBSTR (COLUMN_NAME, 0, 1)) || SUBSTR (REPLACE (INITCAP (COLUMN_NAME), '_', ''), 2) || ',' AS C3
  , '#' || LOWER (COLUMN_NAME) || '#,' AS C4
  , '#' || LOWER (SUBSTR (COLUMN_NAME, 0, 1)) || SUBSTR (REPLACE (INITCAP (COLUMN_NAME), '_', ''), 2) || '#,' AS C5
  , '#{' || LOWER (COLUMN_NAME) || '},' AS C6
  , '#{' || LOWER (SUBSTR (COLUMN_NAME, 0, 1)) || SUBSTR (REPLACE (INITCAP (COLUMN_NAME), '_', ''), 2) || '},' AS C7
  , '<result property="' || LOWER (COLUMN_NAME)
       || '" column="' || COLUMN_NAME || '" />' AS R1
  , '<result property="' || LOWER (SUBSTR (COLUMN_NAME, 0, 1)) || SUBSTR (REPLACE (INITCAP (COLUMN_NAME), '_', ''), 2)
           || '" column="' || COLUMN_NAME || '" />' AS R2
  , 'private ' ||
    CASE
      WHEN DATA_TYPE IN ( 'NUMBER' ) THEN 'int'
      WHEN DATA_TYPE IN ( 'VARCHAR', 'VARCHAR2', 'TEXT', 'CLOB', 'BLOB', 'CHAR' ) THEN 'String'
      WHEN DATA_TYPE IN ( 'DATE', 'DATETIME', 'TIMESTAMP' ) THEN 'Date'
    END || ' ' || LOWER (COLUMN_NAME) || ';' AS V1
  , 'private ' ||
    CASE
      WHEN DATA_TYPE IN ( 'NUMBER' ) THEN 'int'
      WHEN DATA_TYPE IN ( 'VARCHAR', 'VARCHAR2', 'TEXT', 'CLOB', 'BLOB', 'CHAR' ) THEN 'String'
      WHEN DATA_TYPE IN ( 'DATE', 'DATETIME', 'TIMESTAMP' ) THEN 'Date'
    END || ' ' || LOWER (SUBSTR (COLUMN_NAME, 0, 1)) || SUBSTR (REPLACE (INITCAP (COLUMN_NAME), '_', ''), 2) || ';' AS V2
FROM (
  SELECT A.COLUMN_ID
    , A.COLUMN_NAME
    , A.DATA_TYPE
    , B.COMMENTS
  FROM ALL_TAB_COLUMNS A, ALL_COL_COMMENTS B
  WHERE A.TABLE_NAME = B.TABLE_NAME
  AND A.COLUMN_NAME = B.COLUMN_NAME
  AND A.OWNER = B.OWNER
  AND A.OWNER = UPPER('owner name')          /*owner name*/
  AND A.TABLE_NAME = UPPER('table name')      /*table name*/
  ORDER BY A.COLUMN_ID
);

