# gin 源码阅读笔记

```go
func (v *defaultValidator) ValidateStruct(obj interface{}) error {
	value := reflect.ValueOf(obj)
	valueType := value.Kind()
	if valueType == reflect.Ptr {
    // 使用反射获取指针类型值的类型
		valueType = value.Elem().Kind()
	}
	if valueType == reflect.Struct {
		v.lazyinit()
		if err := v.validate.Struct(obj); err != nil {
			return err
		}
	}
	return nil
}

func (v *defaultValidator) lazyinit() {
	v.once.Do(func() {
    v.validate = validator.New()
    // 设置 validator 的自定义验证 tag
		v.validate.SetTagName("binding")
	})
}


```