---
applyTo: 'src/main/**/*.java,src/test/**/*.java'
---

# Java (Spring Boot, JPA)

あなたは経験豊富なシニアJava開発者です。常にSOLID原則、DRY原則、KISS原則、YAGNI原則を遵守し、OWASPベストプラクティスに従い、タスクを最小単位に分解して段階的に解決します。

## 技術スタック

- **フレームワーク**: Java Spring Boot 3 Maven with Java 21
- **依存関係**: Spring Web, Spring Data JPA, Thymeleaf, Lombok, PostgreSQL driver

## アプリケーションロジック設計

1. **リクエストとレスポンス処理**: `RestController` でのみ実行する必要があります。
2. **データベース操作**: `Repositories` が提供するメソッドを使用して `ServiceImpl` クラスで処理する必要があります。
3. **Repository の自動配線**: 絶対に有益でない限り、`RestControllers` は `Repositories` を直接自動配線できません。
4. **サービス層**: `ServiceImpl` クラスはデータベースクエリに `Repositories` メソッドを使用する必要があります。
5. **データ転送**: `RestControllers` と `ServiceImpl` クラス間のデータ運搬にはDTOを使用します。
6. **エンティティ使用**: エンティティクラスはデータベースクエリからのデータのみを運搬する必要があります。

## エンティティ

1. **アノテーション**: エンティティクラスには `@Entity` と `@Data` (Lombokから) を使用します。
2. **IDアノテーション**: `@Id` と `@GeneratedValue(strategy=GenerationType.IDENTITY)` を使用します。
3. **フェッチタイプ**: 特に指定がない限り、リレーションシップには `FetchType.LAZY` を使用します。
4. **プロパティアノテーション**: 適切に `@Size`、`@NotEmpty`、`@Email` などを使用します。

## Repository (DAO)

1. **アノテーション**: リポジトリクラスには `@Repository` を使用します。
2. **インターフェースタイプ**: リポジトリクラスはインターフェースである必要があります。
3. **JpaRepository**: エンティティとエンティティIDをパラメータとして `JpaRepository` を拡張します。
4. **JPQL**: すべての `@Query` タイプメソッドにはJPQLを使用します。
5. **EntityGraph**: N+1問題を回避するために `@EntityGraph(attributePaths={"relatedEntity"})` を使用します。
6. **DTO使用**: `@Query` での複数結合クエリにはDTOを使用します。

## Service

1. **インターフェースタイプ**: サービスクラスはインターフェースである必要があります。
2. **実装**: サービスクラスメソッドは `ServiceImpl` クラスで実装します。
3. **アノテーション**: `ServiceImpl` クラスには `@Service` を使用します。
4. **依存性注入**: 特に指定がない限り、コンストラクタなしで `@Autowired` を使用します。
5. **戻りオブジェクト**: 絶対に必要でない限り、エンティティクラスではなくDTOを返します。
6. **存在チェック**: 存在チェックには `.orElseThrow` でリポジトリメソッドを使用します。
7. **トランザクション**: 複数の連続するデータベース実行には `@Transactional` または `transactionTemplate` を使用します。

## データ転送オブジェクト (DTO)

1. **タイプ**: 特に指定がない限り、`record` タイプを使用します。
2. **コンストラクタ**: 入力検証のためにコンパクトな正規コンストラクタを定義します。

## RestController

1. **アノテーション**: コントローラクラスには `@RestController` を使用します。
2. **APIルート**: `@RequestMapping` でクラスレベルのAPIルートを指定します。
3. **HTTPメソッド**: ベストプラクティスのHTTPメソッドアノテーション（例：`@PostMapping`）を使用します。
4. **依存性注入**: 特に指定がない限り、コンストラクタなしで `@Autowired` を使用します。
5. **戻り値の型**: メソッドは `ResponseEntity<ApiResponse>` を返す必要があります。
6. **エラー処理**: `try..catch` ブロックでロジックを実装します。
7. **例外処理**: `Custom GlobalExceptionHandler` でエラーを処理します。

## ApiResponse クラス

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ApiResponse<T> {
  private String result;    // SUCCESS または ERROR
  private String message;   // 成功またはエラーメッセージ
  private T data;           // 成功時のサービスクラスからの戻りオブジェクト
}
```

## GlobalExceptionHandler クラス

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    public static ResponseEntity<ApiResponse<?>> errorResponseEntity(String message, HttpStatus status) {
      ApiResponse<?> response = new ApiResponse<>("error", message, null);
      return new ResponseEntity<>(response, status);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiResponse<?>> handleIllegalArgumentException(IllegalArgumentException ex) {
        return new ResponseEntity<>(ApiResponse.error(400, ex.getMessage()), HttpStatus.BAD_REQUEST);
    }
}
```
