20230320

Page 객체로 페이지 구현하는 법은 페이지 유틸을 만들어서 사용하던 때랑 비교하면 무진장 편리하다.

jpa repository 에 Page 객체를 만든다.
```java
// Native query 로 할 경우 countQuery 를 사용해야 한다.
Page<PcStatusJoinStatusHistoryDTO> findAll(Pageable pageable);
```

(service는 생략)

```java
public ResponseEntity<?> getPcStatusJoinStatusHistoryByIp(
            @PathVariable String ip,
            @RequestParam(defaultValue = "-1") int page, // 현재 페이지
            @RequestParam(defaultValue = "9") int size // 사이즈
    ) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("index").descending());
        Page<PcStatusJoinStatusHistoryDTO> result = service.findByIp(ip, pageable);

        if (result.isEmpty()) {
            return ResponseEntity.notFound().build();
        } else {
            return ResponseEntity.ok(result);
        }
```

PageRequest 객체를 통해 Pageable 클래스를 선언한다. PageRequest에는 현재 페이지, 사이즈, 정렬 방식을 선택할 수 있다.
Page 의 결과물은 jpa repository의 명령에 만든 pageable 객체를 넣어서 불러온다.


