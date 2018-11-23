<template>
<el-container>

    <!-- 菜单预览 -->

    <el-aside>
        <el-alert type="success" :closable='false' title="菜单预览"></el-alert>
        <el-menu v-if="MainMenu.length" @open="handleOpen" @close="handleClose" @unique-opened="true">
            <label v-for="x,index in MainMenu">
          <!-- 导航类 -->
          <label v-if="x.mode==='1'">
            <el-submenu :index="index+''">
              <template slot="title">
                <i :class="x.icon"></i>
                <span>{{x.title}}</span>
              </template>
              <label v-if="x.children.length">
                <label v-for="y,z in x.children">
                  <el-menu-item :index="index+'-'+z">
                    <i :class="y.icon"></i>
                    <span slot="title">{{y.title}}</span>
                  </el-menu-item>
                </label>
            </label>
            </el-submenu>
            </label>
            <!-- 导航项 -->
            <label v-else-if="x.mode==='2'">
            <el-menu-item :index="index+''">
              <i :class="x.icon"></i>
              <span slot="title">{{x.title}}</span>
            </el-menu-item>
          </label>

            </label>
        </el-menu>
        <label v-else>
        <center>
          <h1>尚未创建菜单。</h1>
        </center>
      </label>
    </el-aside>

    <el-container>
        <!-- 菜单管理头部 -->
        <el-header>
            <el-alert type="success" :closable='false' title="菜单管理"></el-alert>
        </el-header>

        <!-- 菜单编辑 -->
        <el-main style="width:500px;">
            <el-form ref='from' :model="form" label-width="90px">
                <el-form-item label="菜单类型：">
                    <el-radio-group v-model="form.MenuType" :change="childMenu" size="small">
                        <el-radio label="Top" border>顶级菜单</el-radio>
                        <el-radio label="Child" border>子菜单</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="所属菜单：" v-if="form.MenuType!='Top'">
                    <label>{{parentMenu}}</label>
                </el-form-item>
                <el-form-item label="菜单名称：">
                    <el-input v-model="form.title" placeholder="填写名称"></el-input>
                </el-form-item>
                <el-form-item label="排  序：" v-show="false">
                    <el-input v-model="form.number"></el-input>
                </el-form-item>
                <el-form-item label="菜单图标：">
                    <el-input v-model="form.icon"></el-input>
                </el-form-item>
                <el-form-item label="菜单类型：" v-if="form.MenuType==='Top'">
                    <el-radio-group v-model="form.mode" size="mini">
                        <el-radio label="1" border>导航类</el-radio>
                        <el-radio label="2" border>导航项</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="连接页：" v-show="false">
                    <el-input v-model="form.linkto"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="addMenu">添加菜单</el-button>
                </el-form-item>

            </el-form>
        </el-main>
    </el-container>
</el-container>
</template>

<script>
export default {
    data() {
        return {

            form: {
                MenuType: "Top",
                id: null,
                parentid: "0",
                title: '',
                number: '1',
                icon: "el-icon-location",
                linkto: "zongcheng",
                mode: "1"
            },
            MainMenu: [],

        }
    },
    computed: {
        childMenu() {

            if (this.form.MenuType != "Top") {
                this.form.mode = "2"
            } else {
                this.form.mode = "1"
            }

        },
        parentMenu() {
            let parentMenuName = "请选择父菜单！"
            if (this.MainMenu.length && this.form.mode === "2") {
                parentMenuName = this.MainMenu[this.form.parentid].title

            }
            return parentMenuName
        }

    },
    methods: {

        addMenu() {
            if (this.MainMenu.length === 0 && this.form.mode === '2') {
                this.alertMessage('<center>请先创建一个顶级菜单！</center>')
            } else if (this.form.title === '') {
                this.alertMessage('<center>请输入菜单名称！</center>')
            } else {
                let MenuJson
                if (this.form.mode === '1') {
                    MenuJson = {
                        title: this.form.title,
                        number: this.form.number,
                        icon: this.form.icon,
                        mode: this.form.mode,
                        children: []
                    }
                    this.MainMenu.push(MenuJson)

                } else {
                    MenuJson = {
                        parentid: this.form.parentid,
                        title: this.form.title,
                        number: this.form.number,
                        icon: this.form.icon,
                        linkto: this.form.linkto,
                        mode: this.form.mode
                    }
                    if (this.form.MenuType != 'Top') {
                        let child = this.MainMenu[this.form.parentid].children.length
                        this.MainMenu[this.form.parentid].children[child] = MenuJson
                    } else {
                        this.MainMenu.push(MenuJson)
                    }
                }
                this.MainMenu.push()

                console.log(this.MainMenu)

            }
        },
        alertMessage(message) {
            this.$alert(message, '操作提示', {
                dangerouslyUseHTMLString: true
            });
        },
        handleOpen(key, keyPath) {
            this.form.parentid = key

            console.log(this.form.parentid, key, keyPath);
        },
        handleClose(key, keyPath) {
            console.log(key, keyPath);
        }
    }
}
</script>
