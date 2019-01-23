# Coding convention for Yobimi Team
Tài liệu được xây dựa trên [AOSP Java Code Style for Contributors](https://source.android.com/setup/contribute/code-style). 

Các chuẩn code được mô tả dưới đây, rất khuyến khích các thành viên trong team nên tuân theo.

## 1. Java language rules
### 1.1 Không bỏ qua ngoại lệ

Ví dụ dưới đây là 1 trường hợp bỏ qua ngoại lệ:
```java
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) { }
  }
```
Bạn không được làm như vậy. Theo bạn nghĩ thì đoạn code trên sẽ không bao giờ xảy ra ngoại lệ và cũng không quan trọng nếu không xử lý ngoại lệ. Tuy nhiên, một ngày nào đó bạn sẽ gặp vấn đề này. Vì vậy, bạn phải xử lý tất cả các ngoại lệ và xem đó như là một nguyên tắc. Một số cách xử lý ngoại lệ:
* Ném ngoại lệ cho người gọi phương thức của bạn
```java
  void setServerPort(String value) throws NumberFormatException {
      serverPort = Integer.parseInt(value);
  }
```
* Ném ngoại lệ phù hợp với mức độ trừu tượng của bạn
```java
  void setServerPort(String value) throws ConfigurationException {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        throw new ConfigurationException("Port " + value + " is not valid.");
    }
  }
```
* Xử lý lỗi và thay thế một giá trị phù hợp (giá trị mặc định) trong khối `catch() {}`
```java
  /** Set port. If value is not a valid number, 80 is substituted. */

  void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        serverPort = 80;  // default port for server
    }
  }
```
* Xử lý lỗi và ném ra `RuntimeException`. Điều này là nguy hiểm, vì nó sẽ gây ra lỗi ứng dụng. Vì vậy bạn hãy chắc chắn nếu muốn dùng cách này.
```java
  /** Set port. If value is not a valid number, die. */

  void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        throw new RuntimeException("port " + value " is invalid, ", e);
    }
  }
```
* Như một giải pháp cuối cùng, đó là bạn chắc chắn code hoạt động tốt và bỏ qua ngoại lệ. Nhưng bạn phải chú thích rõ ràng tại sao đoạn code hoạt động tốt.
```java
/** If value is not a valid number, original port number is used. */

void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        // Method is documented to just ignore invalid user input.
        // serverPort will just be unchanged.
    }
}
```
### 1.2 Không bắt ngoại lệ chung
Đoạn code dưới đây thể hiện sự lười biếng khi xử lý ngoại lệ:
```java
  try {
      someComplicatedIOFunction();        // may throw IOException
      someComplicatedParsingFunction();   // may throw ParsingException
      someComplicatedSecurityFunction();  // may throw SecurityException
      // phew, made it all the way
  } catch (Exception e) {                 // I'll just catch all exceptions
      handleError();                      // with one generic handler!
  }
```
Bạn không được làm như vậy. Trong hầu hết các trường hợp, không khuyến khích việc bắt ngoại lệ chung hoặc ném ra ngoại lệ.Vì điều này rất nguy hiểm khi không xử lý được ngoại lệ mong muốn để xử lý lỗi. Xem thêm [tại đây](https://source.android.com/setup/contribute/code-style#dont-catch-generic-exception).
