## Задание 5

В задании используется диалект PostgreSQL.

#### Запрос 1.
Для Олимпийских игр 2004 года сгенерируйте список (год рождения, количество игроков, количество золотых медалей), содержащий годы, в которые родились игроки, количество игроков, родившихся в каждый из этих лет, которые выиграли по крайней мере одну золотую медаль, и количество золотых медалей, завоеванных игроками, родившимися в этом году.

```sql
SELECT 
	EXTRACT(YEAR FROM PLAYERS.BIRTHDATE),
	COUNT(DISTINCT PLAYERS.PLAYER_ID),
	COUNT(RESULTS.MEDAL)
FROM PLAYERS
INNER JOIN RESULTS ON PLAYERS.PLAYER_ID = RESULTS.PLAYER_ID
INNER JOIN EVENTS ON RESULTS.EVENT_ID = EVENTS.EVENT_ID
INNER JOIN OLYMPICS ON EVENTS.OLYMPIC_ID = OLYMPICS.OLYMPIC_ID
WHERE MEDAL = 'GOLD'
  AND YEAR = 2004
GROUP BY 1
```

#### Запрос 2.

Перечислите все индивидуальные (не групповые) соревнования, в которых была ничья в счете, и два или более игрока выиграли золотую медаль.

_Предполагаю, что под условием "ничья в счете" имеется в виду, что в соревновании хотя бы два игрока получили одинаковое количество очков result._

```sql
SELECT EVENTS.EVENT_ID
FROM EVENTS
INNER JOIN RESULTS ON EVENTS.EVENT_ID = RESULTS.EVENT_ID
WHERE IS_TEAM_EVENT = 0
  AND MEDAL = 'GOLD'
GROUP BY EVENTS.EVENT_ID
HAVING COUNT(MEDAL) > 1
AND COUNT(DISTINCT RESULTS.RESULT) = 1
```

#### Запрос 3.

Найдите всех игроков, которые выиграли хотя бы одну медаль (GOLD, SILVER и BRONZE) на одной Олимпиаде. (player-name, olympic-id).

```sql
SELECT PLAYERS.NAME, EVENTS.OLYMPIC_ID
FROM PLAYERS
INNER JOIN RESULTS ON PLAYERS.PLAYER_ID = RESULTS.PLAYER_ID
INNER JOIN EVENTS ON RESULTS.EVENT_ID = EVENTS.EVENT_ID
WHERE RESULTS.MEDAL = 'GOLD'
				OR RESULTS.MEDAL = 'SILVER'
				OR RESULTS.MEDAL = 'BRONZE'
```

#### Запрос 4.

В какой стране был наибольший процент игроков (из перечисленных в наборе данных), чьи имена начинались с гласной?

```sql
SELECT LETTERS.COUNTRY_ID
FROM
	(SELECT PLAYERS.COUNTRY_ID,
			COUNT(PLAYERS) AS P_NUM
		FROM PLAYERS
		WHERE LEFT(PLAYERS.NAME, 1) IN ('A', 'E', 'I', 'O', 'U')
		GROUP BY PLAYERS.COUNTRY_ID) AS LETTERS
INNER JOIN
	(SELECT PLAYERS.COUNTRY_ID,
			COUNT(PLAYERS) AS P_NUM
		FROM PLAYERS
		GROUP BY PLAYERS.COUNTRY_ID) AS P_ALL 
ON LETTERS.COUNTRY_ID = P_ALL.COUNTRY_ID
ORDER BY CAST(LETTERS.P_NUM AS decimal) / P_ALL.P_NUM DESC
LIMIT 1
```

#### Запрос 5.

Для Олимпийских игр 2000 года найдите 5 стран с минимальным соотношением количества групповых медалей к численности населения.
```sql
SELECT COUNTRIES.COUNTRY_ID
FROM PLAYERS
INNER JOIN COUNTRIES ON COUNTRIES.COUNTRY_ID = PLAYERS.COUNTRY_ID
INNER JOIN RESULTS ON PLAYERS.PLAYER_ID = RESULTS.PLAYER_ID
INNER JOIN EVENTS ON RESULTS.EVENT_ID = EVENTS.EVENT_ID
INNER JOIN OLYMPICS ON EVENTS.OLYMPIC_ID = OLYMPICS.OLYMPIC_ID
WHERE OLYMPICS.YEAR = 2000
  AND EVENTS.IS_TEAM_EVENT = 1
GROUP BY COUNTRIES.COUNTRY_ID,
		 COUNTRIES.POPULATION
ORDER BY CAST(COUNT(RESULTS.MEDAL) AS decimal) / COUNTRIES.POPULATION
LIMIT 5
```