# Data Processing
1) Lấy dữ liệu vào, bóc tách dữ liệu:
 Để có thể làm được điều này, chúng ta sẽ sử dụng thư viện pandas: `import pandas as pd`. Đây là một thư viện rất phổ biến trong cộng đồng data scientist bởi khả năng import data từ các file format đa dạng như json, csv, Excel..., đồng thời cung cấp rất nhiều phương thức xử lý file theo ý muốn của user subsetting, slicing, filtering, merging, groupBy, re-ordering, and re-shaping... Ở đây chúng ta sẽ import data từ hai file "edge_list.csv" và "vertex_set.csv" bằng hàm "read_csv":
```python 
vx_set = pd.read_csv('vertex_set.csv')
eg_list = pd.read_csv('edge_list.csv')
```
Chúng ta có thể print ra để xem data được lưu vào 1 trong biến "vx_set" hay "eg_list" trông như thế nào:
![image](https://user-images.githubusercontent.com/61876488/102631458-4ee23900-4180-11eb-962d-6cf30a7f21a9.png)

Như các bạn có thể thấy, "vx_set" hay "eg_list" đều là thuộc kiểu dữ liệu Dataframe (một cấu trúc dữ liệu được gắn nhãn hai chiều với các cột và hàng như bảng tính (spreadsheet) hoặc bảng (table)). Vấn đề đặt ra bây giờ là làm sao để tách ra từng columns một trong Dataframe thành từng danh sách các giá trị (value) của mỗi thuộc tính thuộc về đối tượng. Lấy ví dụ Dataframe "vx_set" (hình trên), chúng ta có thể hiểu "vx_set" là một list các [dictionary](https://www.w3schools.com/python/python_dictionaries.asp), trong đó mỗi dictionary gồm các thuộc tính của đối tượng như 'ID', 'Full Name', 'Gender', 'Age' sẽ đóng vai trò là key, cùng với value (ở đây là dạng list của các value!) như [0,1,2,...141,142] hay [Carrie Wood,Craig Lopez,...,Eduardo Domez], Để có thể lấy list của các dictionary đó ra ta làm như sau: 
```python
name = vx_set['Full Name'].values.tolist() #vx_set 
gender = vx_set['Gender'].values.tolist()  #vx_set
age = vx_set['Age'].values.tolist()        #vx_set

source = eg_set['Source'].values.tolist()        #eg_list
target = eg_set['Target'].values.tolist()        #eg_list
msg_count = eg_set['Msg Count'].values.tolist()  #eg_list
```

2) Tạo một đối tượng Graph từ các dữ liệu đã được bóc tách:

a) Sơ lược kiến thức về Graph:
Các bạn học môn Cấu trúc dữ liệu và Giải thuật có lẽ không còn xa lạ gì với Graph (đồ thị). Graph là một cấu trúc dữ liệu được xác định bởi hai thành phần:
- Một node hay còn gọi là một **vertex**(điểm).
- Một cạnh (**edge**) E là một kết nối (connection) có hướng hoặc vô hướng giữa hai điểm trong một graph. Trên một cạnh có thể có trọng số (weight) hoặc không.
  
Python cung cấp cho ta một thư viện rất tiện lợi trong việc vẽ Graph đó là "igraph". Vì 
package chứa thư viện này không có sẵn trong google colab nên ta phải install nó vào rồi mới import.

![image](https://user-images.githubusercontent.com/61876488/102680827-a0c1a800-41ee-11eb-997d-b5c90cb3dbad.png)

b) Tạo đối tượng Graph, thêm điểm và cạnh vào Graph:
Khởi tạo một đối tượng Graph của riêng bạn, ví dụ: `g1  = Graph(directed=True)`. Thuộc tính directed biểu thị tính có hướng hoặc vô hướng của Graph, có thể có hoặc không, tùy vào yêu cầu của bài toán đặt ra. Hiện tại đối tượng Graph này đang hoàn toàn "rỗng", do đó trước tiên chúng ta sẽ thêm các điểm (vertex) vào:
```python 
g1.add_vertices(name,{"age": age,"gender": gender})
```
Tham số name là một danh sách tên những người tham gia trong mạng lưới "message" của chúng ta, được lấy từ thuộc tính 'Full Name' của Dataframe "vx_set" lúc nãy, ta sẽ sử dụng chúng để đặt tên cho các điểm trong Graph. Cùng với đó là tham số attribute (thuộc tính) của mỗi điểm trong Graph thuộc kiểu dictionary, nhờ vậy chúng ta có thể thêm vào "profile" nhiều hơn một tính chất của đối tượng được biểu diển bởi điểm đó, ở đây là age và gender. Sau khi tạo xong chúng ta có thể `list(g1.vs)[0:10]` (in ra 10 điểm đầu tiên của graph) để kiểm tra.
Graph đã có đầy đủ các điểm biểu diễn, tiếp theo chúng ta chỉ cần tạo thêm các cạnh (edge) để nối các điểm kia nữa là xong. Tuy nhiên, việc tạo các điểm khác với việc tạo các cạnh ở chỗ, mỗi "name" sẽ xác định mỗi điểm, còn mỗi cạnh thì lại được xác định bởi các cặp điểm đầu mút như source (nguồn), target (đích). Do đó trước tiên chúng ta cần phải gộp chung 2 list source và target lại với nhau trước:
```python 
st = []
for i in range (len(target)): # len(target) hay len(source) đều như nhau
    st.append((source[i],target[i]))
```
Sau đó chúng ta có thể thêm cạnh vào Graph tương tự như đối với các điểm:
```python
g1.add_edges(st,{'msg count': msg_count})
```
Nếu muốn kiểm tra các cạnh được thêm vào ta dùng: `list(g1.es)[0:10]`
Graph đã được tạo xong, về cơ bản đã có thể đem đi vẽ. Để cho chắc kèo, các bạn có thể kiểm tra lại liệu số lượng các điểm đã đủ chưa, điểm này nối với điểm kia tạo thành các cạnh đã đúng chưa thông qua ma trận kề ([adjacency matrix](https://www.programiz.com/dsa/graph-adjacency-matrix)) của Graph vừa mới tạo:
![image](https://user-images.githubusercontent.com/61876488/102684291-d627bf00-4209-11eb-8066-8cf362657c20.png)

Trong đó các hàng (row) và cột (column) đều biểu diễn các điểm. Giá trị 1 đại diện việc điểm này kề (có kết nối) với điểm kia, còn giá trị 0 thì ngược lại. 
c) Lắp ráp dữ liệu mới từ các thành phần của Graph:
Trong tay chúng ta bây giờ là một chiếc Graph đã "đủ lông đủ cánh", chỉ còn một bước nữa là chúng ta sẽ đến đích: vẽ được một graph hoàn chỉnh ngay chính trên browser của bạn. Toàn bộ công việc đó sẽ được công cụ sigma cùng script của mình giải quyết bằng hết, với điều kiện trước tiên là các bạn phải cho chúng một input chất lượng - một file json chứa dữ liệu hay "profile" của Graph mà bạn vừa tạo.
Trước hết mình sẽ nói sơ qua về file json. **JSON** là chữ viết tắt của **J**avascript **O**bject **N**otation, một định dạng lưu trữ thông tin (thường là về các đối tượng trên một app, một website...) có cấu trúc và chủ yếu được sử dụng để trao đổi dữ liệu giữa server và client. JSON có vẻ ngoài đơn giản, thân thiện với coder, lại có thể đọc, xử lý và ghi file dễ dàng hơn nhiều so với  [XML](https://www.codehub.com.vn/Phan-Biet-XML-va-HTML) nên tính ứng dụng của nó hiện nay rất là phổ biến.

![image](https://user-images.githubusercontent.com/61876488/102712156-62f97800-42f1-11eb-9478-8b24cd261658.png)
  
 Như tên gọi gợi ý phần nào,  [Json ban đầu là một phần của JavaScript](https://codelearn.io/sharing/xu-ly-du-lieu-json-voi-javascript), tuy nhiên hiện nay đã phát triển, có chuẩn riêng của mình, khiến cho không chỉ Java Script mà hầu như bất cứ ngôn ngữ lập trình nào cũng có thể tiếp cận được với nó. Python không phải ngoại lệ, ngôn ngữ này có có nguyên cả một thư viện dành riêng cho file json. Chúng ta có thể sử dụng hàm "dump" trong  thư viện này để ghi đè dữ liệu của Graph mới (g1) lên file "data.json" - lúc này đang chứa thông tin của Graph ở dạng "bản mẫu" như thế này:
```json
{
    "nodes": [
       {
          "id": "n0",
          "label": "A node",
          "x": 0,
          "y": 0,
          "size": 3
       },
       {
          "id": "n1", 
          "label": "Another node",
          "x": 3,
          "y": 1,
          "size": 2
       },
       {
          "id": "n2",
          "label": "And a last one",
          "x": 1,
          "y": 3,
          "size": 1
        }
   ],
   "edges": [
        {
          "id": "e0",
          "source": "n0",
          "target": "n1"
        },
        {
          "id": "e1",
          "source": "n1",
          "target": "n2"
        },
        {
           "id": "e2",
           "source": "n2",
           "target": "n0"
        }
    ]
}
```
Chi tiết về quá trình xử lý dữ liệu (deserialize và serialize) json trong python các bạn có thể xem tại [đây](https://codelearn.io/sharing/xu-ly-json-voi-python-nhu-the-nao). Mình sẽ chỉ giải thích tóm gọn nội dung của file json chứa thông tin về Graph "bản mẫu" này và những gì cần làm với nó. Tạm hiểu là những nội dung bên trong cặp dấu ngoặc nhọn "{...}" ở json khi chuyển qua cho python xử lý sẽ được python coi như một dictionary, với 2 key ở đây là "nodes" và "edges". Mỗi key này đều có value là một list các dictionary nhỏ hơn, mỗi dictionary này chứa các "thông số" về các đối tượng "nodes" và "edges". Chẳng hạn, trong cặp key - value biểu diễn các đối tượng "nodes", chúng ta có 3 dictionary ứng với 3 điểm trong Graph "mẫu" có id lần lượt là n0, n1, n2, label (nhãn) lần lượt là "A node", "Another node", "And a last one", tọa độ (x,y) lần lượt là (0,0), (3,1), (1,3), size (kích cỡ) lần lượt là 3, 2, 1. Tương tự như vậy đối với cặp key - value biểu diễn các đối tượng "edges", do đó Graph "mẫu" khi được hiển thị trên browser trông sẽ như sau (ngoại trừ hệ trục tọa độ Descartes Oxy, cùng với thông tin về các điểm ra là do mình vẽ và chú thích để các bạn dễ hình dung):

![image](https://user-images.githubusercontent.com/61876488/102763153-df509180-43ab-11eb-9def-8d533a140e17.png)

**Lưu ý**: Để hiển thị Graph trên browser như trong hình các bạn vào Visual Studio Code, nhấn CTRL + O rồi chọn folder "visualization" rồi thao tác như hình dưới:

![image](https://user-images.githubusercontent.com/61876488/102763962-fd6ac180-43ac-11eb-8d62-70c270084432.png)

Câu hỏi đặt ra bây giờ là làm sao để thay thế toàn bộ các value (vẫn giữ nguyên các key như id, label, x, y, size...) trong một dictionary như thế này của file "data.json":
```json
       {
          "id": "n0",
          "label": "A node",
          "x": 0,
          "y": 0,
          "size": 3
       }
```
Và nếu thay thế được thì chúng ta sẽ thay thế chúng bằng những value nào của Graph mới (g1)? Trước tiên là về các đối tượng "node", "id" của mỗi node trong list có thể thay bằng chỉ số của mỗi vertex trong g1: `g1.vs[i].index`, "label" thay bằng "name" của mỗi vertex: `g1.vs[i].name`, tương tự như vậy, "id" "source" và "target" của các đối tượng "edge" có thể thay bằng `g1.es[i].index`, `g1.es[i].source` và `g1.es[i].target`, "size" của mỗi node có thể để mặc định tất cả đều là 1. Riêng tọa độ (x,y) của mỗi node sẽ phụ thuộc vào thuật toán vẽ biểu đồ mà ta chọn. Thư viện python-igraph cung cấp rất nhiều hàm generic giúp bạn phân bố các điểm trong một object Graph đến các tọa độ trong không gian 2 chiều, 3 chiều,... đã được quy định trong thuật toán (tất nhiên tên thuật toán ta chọn sẽ nằm trong tên hàm luôn). Các hàm này được gọi chung là ["layout"](https://igraph.org/r/doc/layout_.html), có thể kể một vài cái tên tiêu biểu như layout_fruchterman_reingold, layout_star, layout_grid... Vậy nó có liên quan gì đến việc tạo ra các tọa độ (x,y) của các điểm trong graph "g1" của chúng ta?
Lấy hàm layout_fruchterman_reingold làm ví dụ:

![image](https://user-images.githubusercontent.com/61876488/102803171-74727b00-43ea-11eb-9dab-75607d13f594.png)

Như có thể thấy ở hình trên, `g1.layout_fruchterman_reingold()` trả về cho ta một mảng 2 chiều hay chính xác hơn ở đây là một ma trận n x 2, với n là số điểm có trong 1 object Graph, có bao nhiêu điểm thì sinh ra bấy nhiêu tọa độ. Trong đó cột thứ nhất của ma trận biểu diễn hoành độ (x), cột thứ hai biểu diễn tung độ (y). Đúng cái đang cần! Chúng ta sẽ gán các tọa độ này vào toàn bộ các cặp (x,y) của từng node một. Tổng kết lại những điều đã nói ở trên, ta sẽ có một hàm để tạo ra dữ liệu json của Graph mới như sau:

```python
def  get_json(graph):
    layout = graph.layout_fruchterman_reingold()
    js = {"nodes":  [],  "edges":  []}
    for i in  range(len(list(graph.vs))):
        v = graph.vs[i]
        node = {}
        node['id'] = v.index
        node['label'] = v['name']
        node['x'] = layout[i][0]
        node['y'] = layout[i][1]
        node['size'] = 1
        js['nodes'].append(node)
    for e in graph.es:
        edge = {}
        edge['source'] = e.source
        edge['target'] = e.target
        edge['id'] = e.index
        js['edges'].append(edge)
    return js
js = get_json(g1)
``` 
Sau bước này chúng ta đã có một file json hoàn chỉnh:
![image](https://user-images.githubusercontent.com/61876488/102816695-071e1480-4401-11eb-8668-f43d86f3b13f.png)


Việc còn lại cần làm đó là mở file "data.json" ra, dùng hàm "dump" ghi đè nội dung ở trên vào:
``` python
with open('data.json',  'w+')  as f:
    json.dump(js,f)
```

**Mách nhỏ**: nếu để file json như vậy mà hiển thị lên browser thì nhạt nhẽo quá:v Bởi vì nó sẽ chỉ là một Graph toàn là màu đen ("#000000")! Để Graph có thêm chút màu mè, hoa lá, các bạn có thể thêm `node['color'] = "#<hex code của màu>"` vào  `for i in  range(len(list(graph.vs)))` và `edge['color'] = "#<hex code của màu>"`.  Bạn có thể chọn hex code của màu mà mình thích tại [đây](https://www.google.com/search?q=hex%20color&oq=hex%20&aqs=chrome.3.69i57j69i59l3j69i60l4.4759j0j7&sourceid=chrome&ie=UTF-8).
