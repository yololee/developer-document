# idea-自动生成serialVersionId设置

如果不设置`serialVersionUID`，在序列化，反序列化时会根据对象内的字段来自动生成`serialVersionUID`，如果对象中新增加了一个字段那么`serialVersionUID`将会生成一个新的值，会导致原来在序列化后保存的数据在反序列化到对象时因为`serialVersionUID`不一致导致失败如果自己给对象中设置好`serialVersionUID`后，给对象新增一个字段后再进行反序列化时`serialVersionUID`是一致的不会报错，只是新增的字段为空。这里设置idea自动生成`serialVersionUID`是为了方便快速的生成`serialVersionUID`。

1. 打开`File -> Settings -> Editor -> Inspectinos`
2. 选择`Java -> Serialization issues -> Serializable class without 'serialVersionUID'`，将其勾选即可
3. 光标移动到类上按`Alt+Enter`

