# postMapping 간단 원리

1. form 제출 페이지 조회
    - 등록 폼을 보기 위해 `Get /books/new` 호출
2. 빈(Empty) 객체를 model에 등록 
    - @GetMapping 어노테이션으로 함수 실행 
    - 해당 함수에서는 `model.getAttribute(’book’, new Book())` 로 model 에 등록 
        
        → 이후 바인딩으로 book에 접근 가능  
        
    - 다이렉트된 form.html에서는 `th:object = {$book}` 에서 접근 가능 
3. 폼 제출 (POST 요청)
    - 사용자가 브라우저에서 각 필드를 채우고 http 요청 바디를 제출 
4. 데이터 바인딩 및 객체 생성 
    - 서버는 해당 바디에서 객체의 필드에 해당하는 값을 setter 함수로 값을 연결 (데이터 바인딩)
5. 서비스 호출 및 저장 
    - 저장 메서드를 호출해 repository에 저장

## 코드로 정리해본다면 

전부다 정리하면 헷갈릴 수 있으니까 주요하게 생각했던 부분만 따로 가져와봤다. 

### Controller 
```java
    @GetMapping("/new")
    public String getNewBook(Model model){
        model.addAttribute("bookDto", new BookDto());
        return "/books/form";
    }

    @PostMapping
    public String saveBook(@ModelAttribute BookDto bookDto){
        bookService.createBook(bookDto);
        return "redirect:/books";
    }
```

### form.html
```html
<body>
    <form th:action="@{/books}" method="post" th:object="${bookDto}">
        <input type="text" th:field="*{title}">
        <input type="text" th:field="*{author}">
        <input type="text" th:field="*{isbn}">
        <textarea th:field="*{description}" rows="5"></textarea>
        <button type="submit">저장</button>
        <button type="button" onclick="location.href='/books'">취소</button>
    </form>
</body>
```