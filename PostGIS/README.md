# 공간 데이터 추가하기
- `http://data.seoul.go.kr/` 에서 `서울시 법정구역 읍면동 공간정보` 검색
- 서울시 법정구역 읍면동 공간정보 (좌표계: WGS1984) 데이터 셋에서 Map 탭 선택
- SHP 파일 목록의 `TC_SPBE17_2015_W_SHP.zip` 다운로드 압축 해제
- pgadmin4 실행
- Database -> Create > Database 'Seoul'
- DB 선택 후 Qeury Toool 선택
- `CREATE EXTENSION POSTGIS;` 실행(f5)
- PostGIS Shapefile Importer/Export Manager 실행
- Add File -> 파일 지정
- SRID를 4326으로 지정
- PostGIS connection 설정

```
select * from geometry_columns;

UPDATE geometry_columns set srid=5181 where f_table_name="ot_1_w";

alter table ot_1_w
alter column geom type geometry(POINT, 4326)
using ST_Transform(geom, 4326);

ALTER TABLE TC_SPBE17_2015_W RENAME TO EMD;

SELECT SUM(ST_Area(geom)) FROM EMD;

SELECT EMD_NM, ST_Area(geom)
FROM EMD
ORDER BY SHAPE_AREA DESC;

SELECT RN, ST_Length(geom) as LENGTH
FROM ROAD
ORDER BY LENGTH DESC;

SELECT
	R.RN,
	E.EMD_NM
FROM
	ROAD AS R,
	EMD AS E
WHERE
	E.EMD_NM = '연남동'
AND
	ST_Intersects(R.GEOM, E.GEOM);
	
SELECT
	O.RN,
	E.EMD_NM
FROM
	ROAD AS O,
	EMD AS E
WHERE
	ST_Within(O.GEOM, E.GEOM);
	
SELECT * FROM ROAD;

SELECT
	O.NAME AS STATION,
	E.EMD_NM AS BOUNDARY
FROM
	EMD AS E,
	ROAD AS O
WHERE
	ST_DWithin(E.GEOM, O.GEOM, 100)
ORDER BY STATION ASC;
```