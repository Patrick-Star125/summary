# Python基础技巧

以实现功能为模块，进行记录

## I/O

**记录实验结果**

~~~python
path = os.path.join(os.getcwd(), 'result', 'result_1000_ep{}.txt'.format(epoch))
if os.path.exists(path):
    os.remove(path)
    for i in dataset.create_tuple_iterator():
        predictions = model(i[0])
        predictions = np.round(predictions.asnumpy())
        with open(path, mode='a', encoding='utf-8') as f:
            for j in predictions:
                f.write(str(int(j))+'\n')
~~~

