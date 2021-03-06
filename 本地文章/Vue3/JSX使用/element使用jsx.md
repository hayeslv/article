### 1、实现 input 的 v-model

```jsx
export default defineComponent({
	setup() {
   	const formModel = reactive({
      name: ''
    })
    return { formModel };
  },
  render() {
    return (
    	<el-form>
        <el-form-item label="活动名称">
          <el-input value={this.formModel.name} onInput={value => this.formModel.name = value}></el-input>
        </el-form-item>
      </el-form>
    )
  }
}
```



### 2、el-form

注意 el-form 的 :model 属性，在jsx中是这样写的 `props: {{ model: this.data }}` ，比较特别

```jsx
render(h) {
  const rules = {
      idNumber: [
        {
          pattern: /(^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$)|(^[1-9]\d{5}\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{2}$)/,
          message: '证件号码格式有误！',
          trigger: 'blur'
        }
      ]
    }
	return (
  	<el-form
    	ref="form"
    	props={{ model: this.data }} // 注意:model要改成这样写
  		rules={rules}>
    	<el-form-item label="名称" prop="name" label-width="120px">
        <el-input v-model={this.form.name} style="width: 200px;" placeholder="请输入" clearable></el-input>
        // 或者
        <el-input value={this.form.name} onInput={value => this.form.name = value}></el-input>
      </el-form-item>
    </el-form>
  )
}
```



### 3、ElDialog：使用 .sync 修饰符

原版

```tsx
<el-dialog
	title="详情"
  :visible.sync="visible"
  width="700px"
  append-to-body
>
</el-dialog>
```

使用 jsx

```tsx
// 父组件
export default defineComponent({
  setup() {
    const visible = ref(false)
    // 显示详情
    const dialogShowHandler = (flag) => {
      visible.value = flag
    }
    
    setTimeout(() => { // 两秒后显示dialog
      visible.value = true;
    }, 2000)

    return { visible, dialogShowHandler }
  },
  render() {
    return (
      <ContentWarnInfoDetail 
        visible={this.visible}
        on={{
          ['update:visible']: this.dialogShowHandler
        }}
      />
      // 或者
      <EnableDialog visible={this.enableVisible} on={{ ['update:visible']: flag => this.enableVisible = flag }} />
    )
  }
})
```



```tsx
// 子组件
export default defineComponent({
  props: {
    visible: {
      type: Boolean,
      default: false
    }
  },
  setup(props, {emit}) {
    const handleClose = () => {
      emit('update:visible', false)
    }
    return { handleClose }
  },
  render() {
    return (
      <el-dialog
        title="提示"
        visible={this.visible}
        width="700px"
        append-to-body
        before-close={this.handleClose}>
        <span>这是一段信息</span>
        <span slot="footer" class="dialog-footer">
          <el-button onClick={this.handleClose}>关闭</el-button>
        </span>
      </el-dialog>
    )
  }
})
```

#### slot

```tsx
// 父组件
<CodeTypeDialog v-model:visible={this.visible}/>
```

```tsx
// 子组件
export default defineComponent({
  props: {
    visible: { type: Boolean, default: false },
  },
  emits: ["update:visible"],
  setup(props, { emit }) {
    const methods = {
      close() {
        emit("update:visible", false);
      },
    };
    return { methods };
  },
  render() {
    return (
      <ElDialog
        modelValue={this.visible}
        closeOnClickModal={false}
        appendToBody={true}
        v-slots={{
          footer: () => <div class="dialog-footer">
            <ElButton onClick={this.methods.close}>取消</ElButton>
          </div>,
        }}
      >
        内容
      </ElDialog>
    );
  },
});
```





**实例：el-drawer组件**

```tsx
// 父组件
export default defineComponent({
  setup() {
    let visible = ref(false);
    
    const handleVisible = (flag) => {
      visible.value = flag;
    }
    setTimeout(() => {
      visible.value = true;
    }, 2000);
    return { visible, handleVisible };
  },
  render() {
    return (
      <div class="page-content">
        ...
        <Child 
          visible={ this.visible } 
          on={{ ["update:visible"]: this.handleVisible }} 
        />
      </div>
    );
  },
});
```

```tsx
// 子组件
export default defineComponent({
  props: ["visible"],
  setup(props, ctx) {
    const handleClose = () => {
      ctx.emit("update:visible", false)
    }

    return { handleClose }
  },
  render() {
    return (
      <el-drawer
        title="我是标题"
        visible={this.visible}
        direction="rtl"
        before-close={this.handleClose}>
        <span>我来啦!</span>
      </el-drawer>
    )
  }
})
```



### 4、父子通信

```tsx
// 父组件直接传参，子组件通过props接收，通过 ctx.emit 向父组件传
export default defineComponent({
  props: ["visible"], // 接收参数
  setup(props, ctx) { // 或者 setup(props, { emit })
    const handleClose = () => {
      ctx.emit("update:visible", false); // emit向上传递
    }

    return { handleClose }
  },
}
```



### 5、el-table中用到的 scopedSlots

```tsx
<el-table-column label="操作" width="100" scopedSlots={{ default: (scope) => (
  <div>
    <span style="color: #3786FD; cursor: pointer;" onClick={() => this.tableHandler(scope.row)}>选择</span>
  </div>
) }}>
</el-table-column>
```













