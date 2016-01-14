# CPU and Memory in Nodejs

Reference: http://javascript.info/tutorial/memory-leaks

**Memory Leak**

Memory Leak xảy ra khi bộ nhớ bị chiếm dụng trong quá trình chạy chương trình. Trong quá trình chạy một số object, biến, hàm được cấp phát vùng nhớ khi khởi tạo. Tuy nhiên khi sử dụng xong vì một số nguyên nhân khách quan (vd: nó được trỏ bởi object khác, closure, ...) thì nó chưa được thu hồi. Lâu dần bộ nhớ khả dụng của hệ thống bị thâm hụt và có thể gây nên crash, hiện tượng này được gọi là Memory Leak.

**Nguyên Nhân (trong NodeJS và JavaScript)**

Nguyên nhân chính là do Closures

Ví dụ:
```
function setHandler() {
    var elem = document.getElementById('id');
    elem.onclick = function() {
        // ...
    }
}
```
Phần tử DOM liên kết với hàm thông qua event handler onclick và function liên kết ngược lại với elem thông qua LexicalEnviroment.

<img src="https://viblo.asia/uploads/images/9d74fd9ab2537167edd5d2fb4fdde7a327e59012/91a1724cfc69d33b58df90b3b43a73c056a78eae.png"></img>

Cấu trúc này xuất hiện ngay cả khi không có bất kì dòng code nào trong handler. Đặc biệt là addEventListener/attachEvent cũng tạo liên kết cục bộ.

Vậy muốn xóa phần tử elem ta thường dùng:
```
function cleanUp() {
  var elem = document.getElementById('id')
  elem.parentNode.removeChild(elem)
}
```
Gọi hàm cleanUp xóa elem trong DOM. Vẫn còn một tham chiếu LexialEnvironment.elem 
Nhưng vì không có hàm lồng vào (ví dụ trên là onclick), nên LexialEnvironment sẽ bị xóa. Sau đó elem không thể truy cập và sẽ bị xóa

**More leakage patterns**

Để xem thêm các mẫu code bị leak (More leakage patterns) ta có thể xem tại http://www.javascriptkit.com/javatutors/closuresleak/index3.shtml

**Phát hiện và Phân tích**

1. Javascipt trên client thì ta dùng Chrome Dev Tool để snapshot heap memory
xem thêm tại: https://developer.chrome.com/devtools/docs/javascript-memory-profiling
2. Nodejs trên local:
 2.1 sử dụng webstorm (sử dung v8-profiler): 
    https://www.jetbrains.com/webstorm/help/v8-cpu-and-memory-profiling.html
 2.2 sử dụng memwatch và heapdump
    https://www.npmjs.com/package/memwatch
    https://github.com/bnoordhuis/node-heapdump
  tutorial tai: http://www.nearform.com/nodecrunch/self-detect-memory-leak-node/

**Cách phòng tránh**

1. Trả về null khi không sử dụng
vd:
```
function setHandler() {
    var elem = document.getElementById('id');
    elem.onclick = function() {
        // ...
    }
    elem = null; //This breaks the circular reference
}
```
2. Thêm một hàm khác trong closures
   VD:
```
   document.write("Avoiding a memory leak by adding another closure");
           window.onload=function outerFunction(){
           var anotherObj = function innerFunction() {
               alert("Hi! I have avoided the leak");
           };
           (function anotherInnerFunction(){
               var obj =  document.getElementById("element");
               obj.onclick=anotherObj })();
           };
```
   
   Hàm anotherInnerFunction sẽ tự động được gọi và gán event handler cho obj mà không gây ra vòng lặp tham chiếu (circular references) trong closure của cả 2 hàm.
    
    