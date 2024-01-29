20240119

프로젝트에서 이모지를 사용하려 하는데 mariadb에 입력하려고 하니 에러가 났다.
해결방법은 아래와 같다.

우선 데이터베이스의 문자 세트를 변경한다.
```sql
ALTER DATABASE your_database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;

```
다음 모든 테이블의 문자 세트도 변경한다.
```sql
ALTER TABLE your_table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

```
만약 mysql 8 이하라면 application.yml에서도 아래 내용을 설정해주어야 한다.
```yaml
# For application.yml
spring:
  datasource:
    url: jdbc:mysql://localhost/your_database_name?useUnicode=true&characterEncoding=utf8mb4&serverTimezone=UTC

```
