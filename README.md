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

```
code = ast.Expr(value=ast.Call(
    func=ast.Name('print'),
    args=[ast.Str(s='hello')],keywords=[]))

print(ast.unparse(code))
```
kết quả
```
print('hello')
```
nhưng bây giờ muốn code biến hàm gì đó tùm lum thì sao
thì đây
```
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
```
cách đơn giản nhất là sử dụng ast.Module để ghép lại với nhau 
kết quả
```
a = 'world'
print('hello' + a)
```
