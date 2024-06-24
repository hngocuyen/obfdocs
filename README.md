Trước khi chúng ta vào make ast thì mình sẽ đưa ra một số khái niệm


--- Lớp AST 

--- Hàm cơ bản của ast


`ast.parse` : Dùng để phân tích cấu trúc của source

`ast.dump` : Dùng để biên dịch ast.parse dưới dạng ast

`ast.unparse` : Dùng để đưa cấu trúc ast về source

--- Lớp cho các biểu thức
- `Expr`: Biểu thức chung.
- `BoolOp`: Toán tử logic (and, or).
- `BinOp`: Toán tử nhị phân (cộng, trừ, nhân, chia, ...).
- `UnaryOp`: Toán tử đơn (not, -, +, ...).
- `Lambda`: Biểu thức lambda.
- `IfExp`: Biểu thức if (biểu thức điều kiện).
- `Dict`: Từ điển.
- `Set`: Tập hợp.
- `ListComp`: List comprehension.
- `SetComp`: Set comprehension.
- `DictComp`: Dictionary comprehension.
- `GeneratorExp`: Generator expression.
- `Await`: Biểu thức await.
- `Yield`: Biểu thức yield.
- `YieldFrom`: Biểu thức yield from.
- `Compare`: Biểu thức so sánh.
- `Call`: Lời gọi hàm.
- `FormattedValue`: Giá trị được định dạng (trong f-string).
- `JoinedStr`: Chuỗi nối (trong f-string).
- `Constant`: Hằng số.
- `Attribute`: Thuộc tính của một đối tượng.
- `Subscript`: Truy cập phần tử của một đối tượng (vd. list[index]).
- `Starred`: Biểu thức dấu sao (vd. trong unpacking).
- `Name`: Tên biến.
- `List`: Danh sách.
- `Tuple`: Tuple.
- `Slice`: Phần cắt (vd. trong list slicing).

--- Lớp cho các keyword
- `FunctionDef`: Định nghĩa hàm.
- `AsyncFunctionDef`: Định nghĩa hàm async.
- `ClassDef`: Định nghĩa lớp.
- `Return`: keyword return.
- `Delete`: keyword del.
- `Assign`: keyword gán.
- `AugAssign`: keyword gán mở rộng (vd. +=).
- `AnnAssign`: keyword gán với annotation.
- `For`: Vòng lặp for.
- `AsyncFor`: Vòng lặp async for.
- `While`: Vòng lặp while.
- `If`: keyword if.
- `With`: keyword with.
- `AsyncWith`: keyword async with.
- `Raise`: keyword raise.
- `Try`: keyword try.
- `Assert`: keyword assert.
- `Import`: keyword import.
- `ImportFrom`: keyword import từ một module.
- `Global`: keyword global.
- `Nonlocal`: keyword nonlocal.
- `Expr`: keyword biểu thức.
- `Pass`: keyword pass.
- `Break`: keyword break.
- `Continue`: keyword continue.

--- Toán tử
- `Add`: a + b
- `Sub`: a - `b 
- `Mult`: a * b
- `Div`: a / b
- `Mod`: a % b
- `FloorDiv`: a // b
- `And`: a and b 
- `Or`: a or b
- `Eq`: a == b
- `NotEq`: a != b
- `Lt`: a < b
- `LtE`: a <= b
- `Gt`: a > b
- `GtE`: a >= b
- `Is`: a is b
- `IsNot`: a is not b
- `In`: a in b
- `NotIn`: a not in b

--Cách phân tích mã bằng ast

```python
import ast
code = """
a = 5
print('hello')
print(a)
"""
code = ast.parse(code)
print(ast.dump(code,indent=2))
````
```python
Module(
  body=[
    Assign(
      targets=[
        Name(id='a', ctx=Store())],
      value=Constant(value=5)),
    Expr(
      value=Call(
        func=Name(id='print', ctx=Load()),
        args=[
          Constant(value='hello')],
        keywords=[])),
    Expr(
      value=Call(
        func=Name(id='print', ctx=Load()),
        args=[
          Name(id='a', ctx=Load())],
        keywords=[]))],
  type_ignores=[])

````


ở đây chúng ta không cần quan tâm tới cái `Module(` làm gì mà chỉ việc đi phân tích Name, Call và Constant

Thì đơn giản thôi
đầu tiên có Assgin dùng để gắn , cho target là ast.Name (tức) biến được gắn là a, biến a được gắn với Hằng số có value là 5
thứ hai là Expr để cho biểu thức , cho ast.Name là print và hằng số (constant) là 'hello'
thứ ba vẫn là Expr để cho biểu thức , cho ast.Name là print và hằng số (constant) là a
Nghe phân tích vậy chán đúng không?, giờ thì mình sẽ write một code bằng ast

```python
code = ast.Expr(value=ast.Call(
    func=ast.Name('print'),
    args=[ast.Str(s='hello')],keywords=[]))

print(ast.unparse(code))
```
kết quả
```python
print('hello')
```
nhưng bây giờ muốn code biến hàm gì đó tùm lum thì sao
thì đây
```python
import ast

code = ast.Assign(lineno=0,targets=[ast.Name(id='a', ctx=ast.Store())],
                        value=ast.Constant(value="world"))
code1 = ast.Expr(value=ast.Call(func=ast.Name(id='print', ctx=ast.Load()),
                                     args=[ast.BinOp(left=ast.Str(s='hello'), 
                                                     op=ast.Add(), 
                                                     right=ast.Name(id='a', ctx=ast.Load()))],
                                     keywords=[]))
module = ast.Module(lineno=0,body=[code, code1],type_ignores=[])
print(ast.unparse(module))
```pytho
cách đơn giản nhất là sử dụng ast.Module để ghép lại với nhau 
kết quả
```python
a = 'world'
print('hello' + a)
```
Vậy `lineno` là gì ở trên code này , thật ra thì mình cũng không biết , chỉ là thêm vào là nó không bị lỗi nữa thôi =))))))))

Vậy là đủ sơ sơ qua phân tích cơ bản rồi , giờ chúng ta hãy làm cái phân tích tên biến , string , function , builtins 

```python
import ast


code = """
x = 42
y = "hello world"
z = [1, 2, 3]
def hello():
    return 5
print(y)
print("hello worldd")
str(4)
"""

code = ast.parse(code)

var = []
tenham = []
_builtins = []
string = []
_int = []
for i in ast.walk(code):
    if type(i) == ast.Assign: #Như chúng ta đã phân tích thì một ast.Name tức biến sẽ ở trong ast.Assgin, chúng ta kiểm tra trước rồi mới duyệt vào trong
        for j in i.targets:
            if type(j) == ast.Name: #Kiểm tra xem có phải là biến không
                var.append(j.id) #id ở đây thực tế là ast.Name(id=" gì gì đó là tên biến ở đây")
    if type(i) == ast.FunctionDef: #cái này thì dễ lấy chỉ việc lấy tên của nó ra thôi
        tenham.append(i.name)
    if type(i) == ast.Call:
        if type(i.func) == ast.Name:
            ten = i.func.id
            if ten in vars(__builtins__):
                _builtins.append(ten)
    if type(i) == ast.Constant and type(i.value) == str: #Kiểm tra xem có phải là constant và giá trị có phải là str hay  không thì add vào
        string.append(i.value)
    if type(i) == ast.Constant and type(i.value) == int: #Kiểm tra xem có phải là constant và giá trị có phải là int hay  không thì add vào như bên trên thoi thay mỗi hàm
        _int.append(i.value)

for i in var:
    print("Biến : ",i)
for i in tenham:
    print("Hàm : ",i)
for i in string:
    print("string : ",i)
for i in _int:
    print("int : ",i)
for i in _builtins:
    print("hàm builtins : ",i)
```
kết quả
```python
Biến :  x
Biến :  y
Biến :  z
Hàm :  hello
string :  hello world
string :  hello worldd
int :  42
int :  1
int :  2
int :  3
int :  5
int :  4
hàm builtins :  print
hàm builtins :  print
hàm builtins :  str
```


