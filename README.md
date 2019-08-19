# Buil web application with Go

Như chúng ta đã biết, ngày nay ngôn ngữ lập trình Go (Golang) ngày càng trở nên phổ biến, và thông dụng, được nhiều lập trình viên biết tới, và sử dụng nó. Rất nhiều dự án sử dụng Go và đã thành công và phát triển mạnh mẽ. Golang đặc biệt mạnh hơn các ngon ngữ khác khi dùng nó để code cho phần server. Với những dữ liệu lớn và xử lý phức tạp, thì tôi dám khẳng định rằng không ngôn ngữ lập trình nào có thể vượt qua được Golang. Chúng ta chỉ quan tâm, cũng như lựa chọn Go khi làm server mà quên mất rằng có thể sử dụng Go để xây dựng một ứng dụng web, mà chất lượng không khác nhưng ngôn ngữ khác là bao. Bài viết này tôi xin giới thiệu với quý bạn đọc phương pháp xử dụng Golang để xây dựng một ứng dụng web. Để nếu có cơ hội các bạn hãy thử áp dụng với chính dự án của mình để trải nghiệm sự tuyệt vời của Go.

Điều kiện tiên quyết cho quý bạn đọc có thể hiểu được bài viết này là các bạn cần phải nắm được cơ bản về ngôn ngữ lập trình Golang

Mục tiêu của bải viết:

- Hiển thị danh sách các loài chim, với tên và chi tiết về loại chim đó

- Cho phép người dùng đăng một thông tin loài chim mới mà họ tìm thấy

Ứng dụng sẽ có 3 phần:

- phần server

- phần client

- database

## Cài đặt môi trường

1. Cài đặt Gopath

Chạy lệnh 

```
echo $GOPATH
```

trên terminal của bạn, nếu không có kết quả nào được trả về thì chứng tỏ bạn chưa cài đặt GOPATH trước đó, để cài đặt chạy lệnh

```
export GOPATH="/location/of/your/gopath/directory"
```

2. Tạo thư mục chứa project

Tạo thư mục bằng lệnh sau

```
# Inside the Go directory
$ mkdir src
$ mkdir pkg
$ mkdir bin
```
Chức năng của 3 thư mục lần lượt như sau:

```
bin - chứa tất cả những file được biên dịch từ file code của bạn
pkg - Chứa thư viện cần thiết
src - Chứa những file code của bạn
```

3. Tạo thư mục

Các thư mục dự án bên trong src phải có tên tương ứng như repo của bạn trên github. Ví dụ trên github tôi tạo một repo mới với tên 'bird-demo' lúc đó đường dẫn tới repo của tôi trên github sẽ là 'github.com/nguyenvancanh/bird-demo' thì khi đặt project trong đường dẫn GOPATH cũng phải có tên là: '$GOPATH/src/github.com/nguyenvancanh/bird-demo'

## Code HTTP Server

Tạo 1 file với tên main.go bên trong thư mục project của bạn và thêm đoạn code sau

```

package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", handler)

	http.ListenAndServe(":8080", nil)
}

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World!")
}
```
Tạm thời, chúng ta chỉ demo đơn giản để kiểm tra xem web sẽ chạy  như thế nào, sau khi có đoạn code trên, chạy lệnh sau:

```
go run main.go
```
rồi truy cập vào đường dẫn 

```
 http://localhost:8080
```

trên trình duyệt của bạn, bạn sẽ thấy dòng chữ "Hello World!" trên màn hình.

## Make Route

Để xử dụng được route trong Golang, chúng ta cần cài thêm packate (https://github.com/gorilla/mux)[https://github.com/gorilla/mux] cách xử dụng route như sau

```
package main

import (
	"fmt"
	"net/http"

	"github.com/gorilla/mux"
)

func main() {
	r := mux.NewRouter()

	r.HandleFunc("/hello", handler).Methods("GET")

	http.ListenAndServe(":8080", r)
}

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World!")
}
```

## Testing

Để test cho file main.go ở trên, bạn hãy tạo file main_test.go với nội dung như sau

```
//main_test.go

package main

import (
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestHandler(t *testing.T) {
	req, err := http.NewRequest("GET", "", nil)

	if err != nil {
		t.Fatal(err)
	}

	recorder := httptest.NewRecorder()

	hf := http.HandlerFunc(handler)

	hf.ServeHTTP(recorder, req)

	// Check the status code is what we expect.
	if status := recorder.Code; status != http.StatusOK {
		t.Errorf("handler returned wrong status code: got %v want %v",
			status, http.StatusOK)
	}

	// Check the response body is what we expect.
	expected := `Hello World!`
	actual := recorder.Body.String()
	if actual != expected {
		t.Errorf("handler returned unexpected body: got %v want %v", actual, expected)
	}
}
```

Chạy file test với lệnh

```
go test ./...
```

## Tạo file HTML

Nói cách khác, tới bước này chúng ta sẽ tạo giao diện cho ứng dụng của mình, Để dễ dàng quản lý code, hãy tạo mới 1 thư mục có tên 

```
mkdir assets
```
trong thư mục project của bạn. Tiếp theo, tạo file html đầu tiên của app

```
touch assets/index.html
```
Cập nhật lại route 1 chút

```
func newRouter() *mux.Router {
	r := mux.NewRouter()
	r.HandleFunc("/hello", handler).Methods("GET")

	staticFileDirectory := http.Dir("./assets/")
	staticFileHandler := http.StripPrefix("/assets/", http.FileServer(staticFileDirectory))
	r.PathPrefix("/assets/").Handler(staticFileHandler).Methods("GET")
	return r
}
```

Chạy lại lệnh 

```
go run main.go
```
rồi truy cập vào link (http://localhost:8080/assets/)[http://localhost:8080/assets/] trên trình duyệt của bạn

Tới đây, chúng ta đã có khái niệm cơ bản về việc chạy go trên web, bây giờ sẽ tập trung vào bài toán ở đầu bài viết chúng ta có để cập

## Tạo giao diện

```
<!DOCTYPE html>
<html lang="en">

<head>
 <title>The bird encyclopedia</title>
</head>

<body>
  <h1>The bird encyclopedia</h1>
  <table>
    <tr>
      <th>Species</th>
      <th>Description</th>
    </tr>
    <td>Pigeon</td>
    <td>Common in cities</td>
    </tr>
  </table>
  <br/>

  <form action="/bird" method="post">
    Species:
    <input type="text" name="species">
    <br/> Description:
    <input type="text" name="description">
    <br/>
    <input type="submit" value="Submit">
  </form>

  <script>
    birdTable = document.querySelector("table")
    fetch("/bird")
      .then(response => response.json())
      .then(birdList => {
        //Once we fetch the list, we iterate over it
        birdList.forEach(bird => {
          row = document.createElement("tr")

          species = document.createElement("td")
          species.innerHTML = bird.species
          description = document.createElement("td")
          description.innerHTML = bird.description

          // Add the data elements to the row
          row.appendChild(species)
          row.appendChild(description)
          // Finally, add the row element to the table itself
          birdTable.appendChild(row)
        })
      })
  </script>
</body>
```
Nhìn lại một chút, thì với đoạn code HTML trên, chúng ta có một table để  hiển thị danh sách các loại chim với những thuộc tính của nó. Đồn thời có thêm 1 form submit, cho phép ngưởi dùng submit lên với một loại chim mới mà họ nhìn thấy.Đoạn xử lý script chỉ để thêm giúp chúng ta cập nhật lại danh sách trên trang mà k cần load lại trang.

Phần hiển thị đã xong, bây giờ cần làm API nữa:

- GET /bird - phương thức này dùng để lấy dữ liệu trong database trả về để hiển thị lên danh sách
- POST /bird - phương thức này sẽ thêm mới 1 loại chim vào danh sách có sẵn

Tạo mới một file bird_handles.go cùng cấp với file main.go bên trên
