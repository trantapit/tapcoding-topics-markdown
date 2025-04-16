# GraphQL - Tìm hiểu về GraphQL

### Table of Contents
- [GraphQL là gì?](#graphql-là-gì)
- [Query](#query)
- [Mutation](#mutation)
- [Schemas](#schemas)
- [Kết luận](#kết-luận)


## GraphQL là gì?

Như các bạn đã biết, GraphQL ngày càng được sử dụng nhiều trong các ứng dụng, và rất nhiều developer đã chuyển sang GraphQL vì những tính năng tuyệt vời mà nó cung cấp. Vậy GraphQL là gì?

GraphQL là ngôn ngữ truy vấn mạnh mẽ cho API, cho phép khách hàng yêu cầu chính xác dữ liệu họ cần. Nói cách khác, nó cung cấp một interface chung giữa client và server cho việc lấy và thao tác với dữ liệu. Điều này thúc đẩy sự rõ ràng, nhất quán và hợp tác giữa các nhóm.
http://graphql.org

Nói cách khác, GraphQL là một đặc tả và ngôn ngữ truy vấn dữ liệu theo kiểu khai báo. Nó được tạo ra nhằm cung cấp một giao diện chung giữa client (máy khách) và server (máy chủ) để truy xuất và xử lý dữ liệu.

Từ cái tên GraphQL, bạn có thể nghĩ rằng GraphQL liên quan đến graph database hay SQL. Thực tế GraphQL không liên quan gì đến data storage hay graph database.

Để hiểu hơn về GraphQL là gì, hãy xem những tiện ích mà nó cung cấp:

- **Declarative**: Với GraphQL, bạn tự định nghĩa ra những dữ liệu bạn cần.
- **Hierarchical**: Với mỗi request, bạn có thể lấy được một object và cả những object liên quan với object đó, ví dụ một `Author` cùng với các `Posts` mà người đó tạo ra, mà các `Comments` trên mỗi Post.
- **Strongly-typed**: Với hệ thống GraphQL, ta có thể mô tả dữ liệu có thể truy vấn từ server dựa trên kiểu của object và dữ liệu trả về sẽ phù hợp với kiểu object chỉ định trong câu truy vấn.
- **Not Language specific**: GraphQL không gắn với một ngôn ngữ lập trình cụ thể nào cả.
- **Compatible with any backend**: GraphQL không bị giới hạn bởi một data storage cụ thể; bạn có thể sử dụng data và code có sẵn, kể cả kết nối đến third-party APIs.
- **Introspective**: một GraphQL server có thể được truy vấn về schema cụ thể của nó.

Thêm vào đó, GraphQL rất để sử dụng vì nó có cú pháp giống JSON và cho rất nhiều lợi ích về performance.

Tiếp theo, ta sẽ đi tìm hiểu về cú pháp và các hoạt động có thể thực thi với GraphQL.


## Query

Query được sử dụng để thực thi hành động đọc, lấy dữ liệu từ server. Dưới đây là một ví dụ về một query đơn giản và response tương ứng:

```graphql
// Basic GraphQL Query
{
  author {
    name
    posts {
      title
    }
  }
}
```

```json
// Basic GraphQL Query Response
{
  "data": {
    "author": {
      "name": "Tap Coding",
      "posts": [
        {
          "title": "How to build a collaborative note app using Laravel"
        },
        {
          "title": "Event-Driven Laravel Applications"
        }
      ]
    }
  }
}
```

Đây là một query đơn giản để lấy `name` của một `author` cùng với các `posts` tương ứng của `author` đó. Chú ý rằng query và response có cấu trúc giống nhau

Bây giờ ta sẽ tìm hiểu về các thành phần của một GraphQL query

Ở câu query trên, ta có thể không dùng từ khóa `query`. Nếu một operation không chỉ định type thì mặc định GraphQL sẽ cho rằng operation đó là một `query`. Một query có thể có tên (GetAuthor). Mặc dù tên của query có thể có hoặc không nhưng nó sẽ giúp dễ hiểu query đó dùng để làm gì.

```graphql
query GetAuthor {
  author {
    # The name of the author
    name
  }
}
```

### Fields

fields là thành phần cơ bản của một object mà ta muốn thu được từ server. Ở câu query trên, `name` là một field của author

### Arguments

Giống như các hàm trong các ngôn ngữ lập trình có thể chấp nhận các tham số truyền vào, một `query` có thể chấp nhận các tham số. Tham số truyền vào có thể là tùy chọn hoặc bắt buộc. Vì vậy, chúng ta có thể viết lại truy vấn của mình để chấp nhận ID của tác giả như một tham số như dưới đây:
Một query có thể nhận tham số truyền vào.

```graphql
{
  author(id: 5) {
    name
  }
}
```

**Lưu ý:** GraphQL bắt buộc strings phải được bọc trong dấu nháy kép.

### Variables

Bên cạnh tham số, một query cũng có thể có các biến. Các biến được đặt trước bởi dấu `$` và theo sau là kiểu của nó.

```graphql
query GetAuthor($authorID: Int!) {
  author(id: $authorID) {
    name
  }
}
```

Một biến cũng có thể có giá trị mặc định của nó.

```graphql
query GetAuthor($authorID: Int! = 5) {
  author(id: $authorID) {
    name
  }
}
```

Giống với tham số, biến cũng có thể là tùy chọn hoặc bắt buộc. Ở ví dụ trên `$authorID` là bắt buộc vì sử dụng ký hiệu `!` khi định nghĩa biến.

### Aliases

Với query ở trên, giả sử chúng ta muốn có được các tác giả có ID 5 và 7 tương ứng. Chúng ta có thể muốn làm điều gì đó như thế này:

```graphql
{
  author(id: 5) {
    name
  }
  author(id: 7) {
    name
  }
}
```

Điều này sẽ gây ra một lỗi vì sẽ có xung đột giữa hai trường `name`. Để giải quyết điều này, chúng ta sẽ sử dụng Aliases. Với Aliases, chúng ta có thể đưa ra các trường tùy chỉnh tên và yêu cầu dữ liệu từ cùng một trường với các tham số khác nhau.

```graphql
{
  tap: author(id: 5) {
    name
  }
  coding: author(id: 7) {
    name
  }
}
```

Và response sẽ có dạng:

```json
# Response
{
  "data": {
    "tap": {
      "name": "Tran Tap"
    },
    "coding": {
      "name": "Developer"
    }
  }
}
```

### Fragments

Fragments are là tập hợp các trường có thể tái sử dụng có thể được bao gồm trong các truy vấn khi cần thiết. Giả sử chúng ta cần lấy field `TwitterHandle` trên đối tượng `author`, chúng ta có thể dễ dàng làm điều đó với:

```graphql
{
  tap: author(id: 5) {
    name
    twitterHandle
  }
  coding: author(id: 7) {
    name
    twitterHandle
  }
}
```

Nhưng nếu chúng ta muốn lấy nhiều fields hơn? Điều này có thể trở nên lặp đi lặp lại và dư thừa. Đó là lý do fragments xuất hiện. Dưới đây là cách chúng ta sẽ giải quyết tình huống trên bằng fragments:

```graphql
{
  tap: author(id: 5) {
    ...authorDetails
  }
  coding: author(id: 7) {
    ...authorDetails
  }
}

fragment authorDetails on Author {
  name
  twitterHandle
}
```

Với query trên, khi cần thêm một field nào đó, ta chỉ cần thêm vào fragment.

### Directives

Directives cho phép ta thay đổi cấu trúc query một cách linh hoạt sử dụng các biến. GraphQL có 2 directive:

<ul>
  <li>`@include` sẽ include một field hay fragment khi tham số `if` argument là `true`</li>
  <li>`@skip` sẽ bỏ qua một field hay fragment khi tham số `if` argument là `true`</li>
</ul>

Cả hai directive nhận tham số kiểu boolean.

```graphql
query GetAuthor($authorID: Int!, $notOnTwitter: Boolean!, $hasPosts: Post) {
  author(id: $authorID) {
    name
    twitterHandle @skip(if: $notOnTwitter)
    posts @include(if: $hasPosts) {
      title
    }
  }
}
```

## Mutation

Trong việc sử dụng API điển hình, có những tình huống mà chúng ta muốn sửa đổi dữ liệu trên server. Đó là lý do `mutations` xuất hiện. `mutations` được sử dụng để thực hiện các hoạt động write. Bằng cách sử dụng các `mutations`, chúng ta có thể đưa ra yêu cầu đến server để sửa đổi hoặc cập nhật dữ liệu cụ thể và chúng ta sẽ nhận được response có chứa các bản cập nhật được thực hiện. Nó có một cú pháp tương tự như query với một sự khác biệt nhỏ.

```graphql
mutation UpdateAuthorDetails($authorID: Int!, $twitterHandle: String!) {
  updateAuthor(id: $authorID, twitterHandle: $twitterHandle) {
    twitterHandle
  }
}
```

Ta gửi dữ liệu như một payload trong mutation.

```json
// Update data
{
 "authorID": 5,
 "twitterHandle": "tapcoding"
}
```

Sau đó server sẽ trả về respone sau khi thực hiện update:

```json
// Response after update
{
  "data": {
    "id": 5,
    "twitterHandle": "tapcoding"
  }
}
```

Trong response trả về sẽ chữa dữ liệu đã được update.

Một điểm khác biệt quan trọng giữa mutation và query là mutation thực hiện một cách tuần tự còn query có thể thực thi song song.


## Schemas

Schemas mô tả cách dữ liệu được tổ chức và những dữ liệu nào có thể được truy vấn. Schemas cung cấp các kiểu object được sử dụng. GraphQL schema quy định kiểu chặt chẽ, mọi object trong schema phải được chỉ định kiểu. Kiểu được sử dụng để xác định một truy vấn có hợp lệ hay không.

Schemas được xây dựng sử dụng GraphQL schema language. Ví dụ:

```graphql
type Author {
  name: String!
  posts: [Post]
}
```

Schema trên định nghĩa một object kiểu `Author` vơi 2 field là `name` và `posts`. Các field trên một kiểu object có thể tùy chọn hoặc bắt buộc. Ở ví dụ trên field `name` là bắt buộc vì có ký hiệu `!`.

### Arguments

các field trong schema chấp nhận tham số. Các tham số này có thể tùy chọn hoặc bắt buộc (xác định bởi ký hiệu `!`).

```graphql
type Post {
  allowComments(comments: Boolean!)
}
```

### Scalar Types

GraphQL có những scalar type là:

<ul>
  <li>Int: a signed 32‐bit integer</li>
  <li>Float: a signed double-precision floating-point value</li>
  <li>String: A UTF‐8 character sequence</li>
  <li>Boolean: true or false</li>
  <li>ID: represents a unique identifier</li>
</ul>

Các field có kiểu scalar thì không thể chứa các field khác. Ta cũng có thể chỉ định kiểu scalar custom sử dụng từ khóa `scalar`. Ví dụ ta có thể định nghĩa kiểu Date:

```graphql
scalar Date
```

### Enumeration types

Kiểu enumeration là một kiểu đặc biệt của scalar, giới hạn trong một tập các giá trị cho phép. Nó cho phép validate các tham số của kiểu này phải là một trong các giá trị trong tập cho phép, quy định giá trị của một field sẽ luôn là một trong các giá trị của một tập hữu hạn.

Một định nghĩa enum trong GraphQL schema language có dạng:

```graphql
enum Media {
  Image
  Video
}
```

### Input types

Kiểu input được sử dụng trong trường hợp mutation, khi bạn muốn tạo một object. Trong GraphQL schema language, kiểu input giống hoàn toàn với các kiểu thông thường khác, ngoại trừ dùng từ khóa `input` thay vì `type`. Ví dụ:

```graphql
# Input Type
input CommentInput {
  body: String!
}
```

Những field của object kiểu input cũng có thể là những object kiểu input. Object kiểu input không thể chứa tham số trong các field của nó

## Kết luận

Sử dụng GraphQL giúp việc thiết kế API một cách có rõ ràng hơn và giúp dễ dàng xây dựng các truy vấn linh hoạt, hiệu quả hơn. Bằng cách định nghĩa rõ ràng schema và các resolver, bạn cho phép các nhóm frontend và backend có thể hợp tác hiệu quả và độc lập với nhau. Khi được kết hợp với các công cụ hiện đại và những phương pháp tốt nhất, bạn có thể xây dựng các API GraphQL có khả năng mở rộng và dễ bảo trì.

Để tìm hiểu thêm về GraphQL, hãy xem [documentation](https://graphql.org/learn/) và kiểm tra thêm các thông số kỹ thuật của GraphQL. Bạn cũng có thể trao đổi với tôi qua email trantrap.it6491@gmail.

