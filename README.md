# nested_el-checkbox
element-ui的el-checkbox实现嵌套多选，单选

tips:chrome获取本地json数据时会产生跨域问题，建议用firefox直接打开


## 效果图


![](https://github.com/liulinfen/images/blob/master/20190213112402596.gif)

主要功能：
1. 实现多选框层级嵌套
2. 当选中一个二级子菜单的时候，其对应的上级菜单也选中
3. 若二级子菜单无一个选中，则对应的一级菜单也不是选中状态
4. 当一级菜单发生点击事件时，若原本为选中状态，则取消选中，若该菜单下有二级子菜单，所有子菜单也取消选中状态；若原本为未选中状态，则改为选中状态，若该菜单下有二级子菜单，所有子菜单也改为选中状态。
实现已有的选择记录的复现。


## 实现过程

### 1.data数据
	
```
data() {
	return {
		menu:  [] //所有的菜单数组，
		menusIds: [] //已选的菜单id数组
	}
}
```
 ### 2.html部分
 
 &nbsp;&nbsp;要实现一个嵌套的效果，每一个一级菜单应该是一个单独的模块，二级菜单包裹在一级菜单下面，实现一个附属的效果，添加一个缩进，具体代码如下：

```
	 <div class="checkbox-table" v-for="(item, indexkey) in menu" :key="item.id">
        <template>
          <el-checkbox class="check1" style="font-weight: 600;margin-bottom: 15px " v-model='menusIds' :label="item.id" @change='handleCheck(1,indexkey)'>
            {{item.name}}
          </el-checkbox>
          <div style="margin-bottom: 20px;">
            <div v-for="list in item.children" class="line-check" :key="list.id" style="display: inline-block; margin-left: 20px;margin-bottom: 20px;" >
              <el-checkbox class="check2" @change='handleCheck(2,indexkey)' v-model="menusIds" :label="list.id">
                {{list.name}}
              </el-checkbox>
            </div>
          </div>
        </template>
      </div>
```

### 3.点击事件的处理
`提示:element-ui的el-checkbox默认change事件在这里千万不能使用，不适合这个使用场景，循环嵌套的change事件必须要自定义`。
&nbsp;&nbsp;自定义的change事件必须标明是哪个层级发生了点击事件。
&nbsp;&nbsp;handlecheck(type, a = 0)；第一个参数是来表示哪个层级发生了点击事件。（如果是多层级的话，这个函数的参数还要多增加一些，要标明当前点击的是哪个层级，及他对应的所有父祖层级都应该用参数标明。）
&nbsp;&nbsp;把所有选中的菜单的id放到一个数组里面，作为el-checkbox的model数据，只有当着el-checkbox的id在menusIds里时，才会有选中的状态，否则是未选中的。依照这个方法，我们只要在点击这个某个菜单时，判断他是否在menusIds里存在，若存在，则剔除，不存在则把这个id放进去。但是这个过程中有两个注意点：

> 1 若是二级子菜单的点击，要去判断该父级下是否还有二级子菜单是选中的状态，若无，则要顺带把该父级菜单的id也从menusIds中剔除。
> 2. 若是父级的点击，若原来是选中的状态，剔除该父级菜单id的同时，还要剔除其下面所有子集菜单的id；若原来是未选中的，则点击之后添加完该父级菜单的id，还要遍历其下面所有的子集，并把他们的id添加进menusIds。

具体代码如下：

```
// 处理选择事件
      handleCheck(type, a = 0) { // 多选框
        let self = this;
        if (type == 2) { // 二级菜单点击
          let index = 0;
          self.menu[a].children.map(item => {
            if (self.menusIds.indexOf(item.id) > -1) {
              index += 1;
            }
          })
          if (index > 0) {
            if (self.menusIds.indexOf(self.menu[a].id) < 0) {
              self.menusIds.push(self.menu[a].id);
            }
          } else {
            if (self.menusIds.indexOf(self.menu[a].id) > 0) {
              self.menusIds.splice(self.menusIds.indexOf(self.menu[a].id), 1);
            }
          }
        } else {
          if (self.menusIds.indexOf(self.menu[a].id) > -1) {
            self.menu[a].children.map(item => {
              if (self.menusIds.findIndex((n) => n == item.id) < 0) {
                self.menusIds.push(item.id)
              }
            })
          } else {
            self.menu[a].children.map(item => {
              if (self.menusIds.findIndex((n) => n == item.id) > -1) {
                self.menusIds.splice(self.menusIds.findIndex((n) => n == item.id), 1);
              }
            })
          }
        }
      }
```

### 4.已有选择记录的复现
	第三小部分已经说得很清楚了，复现已选择的记录，只要把已选的id放进menusIds，就能实现选中和未选中状态的复现了。
	this.menusIds = params.ids;
