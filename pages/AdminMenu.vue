<template>
<el-container>

    <!-- 菜单预览 -->

    <el-aside>
        <el-alert type="success" :closable='false' title="菜单预览"></el-alert>
        <el-menu name='mainMenues' v-if="MainMenu.length" @open="handleOpen" @select="handleSelece" @close="handleClose" @unique-opened="true">
            <label v-for="x,index in MainMenu">
          <!-- 导航类 -->
          <label v-if="x.mode==='1'">
            <el-submenu :index="index+''">
              <template slot="title">
                <i :class="'icon font_family '+ x.icon"></i>
                <span>{{x.title}}</span>
              </template>
              <label v-if="x.children.length">
                <label v-for="y,z in x.children">
                  <el-menu-item :index="index+'-'+z">
                    <i :class="'icon font_family '+y.icon"></i>
                    <span slot="title">{{y.title}}</span>
                  </el-menu-item>
                </label>
            </label>
            </el-submenu>
            </label>
            <!-- 导航项 -->
            <label v-else-if="x.mode==='2'">
            <el-menu-item :index="index+''">
              <i :class="'icon font_family '+x.icon"></i>
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
                <el-form-item label="操作类型：">
                    <el-radio-group v-model="OperationType" :change="OperationSelect" size="small">
                        <el-radio label="1" border>添加菜单项</el-radio>
                        <el-radio label="2" border v-if="MainMenu.length">修改菜单项</el-radio>
                        <el-radio label="3" border v-if="MainMenu.length">删除菜单项</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="菜单类型：">
                    <el-radio-group v-model="form.MenuType" :change="childMenu" size="small">
                        <el-radio label="Top" v-if="OperationType==='1'||form.MenuType==='Top'" border>顶级菜单</el-radio>
                        <el-radio label="Child" v-if="OperationType==='1'||form.MenuType==='Child'" border>子菜单</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="所属菜单：" v-if="form.MenuType!='Top'">
                    <label>{{parentMenu}}</label>
                </el-form-item>
                <el-form-item label="菜单名称：">
                    <el-input v-model="form.title" placeholder="填写菜单名称"></el-input>
                </el-form-item>
                <el-form-item label="排  序：" v-show="false">
                    <el-input v-model="form.number"></el-input>
                </el-form-item>
                <el-form-item label="菜单图标：">
                    <el-input v-model="form.icon"></el-input>
                    <a href="iconfont/AdminIcon" target="_balck">打开预览</a>
                </el-form-item>
                <el-form-item label="菜单类型：" v-if="form.MenuType==='Top'">
                    <el-radio-group v-model="form.mode" size="mini">
                        <el-radio v-if="OperationType==='1'||form.mode==='1'" label="1" border>导航类</el-radio>
                        <el-radio v-if="OperationType==='1'||form.mode==='2'" label="2" border>导航项</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="连接页：" v-show="false">
                    <el-input v-model="form.linkto"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button v-if="OperationType==='1'" type="primary" @click="addMenu">添加菜单</el-button>
                    <el-button v-if="OperationType==='2'" type="primary" @click="modifyMenu">修改菜单</el-button>
                    <el-button v-if="OperationType==='3'" type="primary" @click="deleteMenu">删除菜单</el-button>
                </el-form-item>
                <el-form-item>
                    <hr style="margin:20px 0px;border:0.1em #dcdee6 solid">
                    </hr>
                    <el-button type="warning" @click="LoadMenu">重新载入菜单</el-button>
                    <el-button type="success" @click="SaveMenu">保存菜单</el-button>
                    <el-alert title="保存当前菜单到数据库中，保存会覆盖原有菜单，请慎重。直接关闭当前页面，当前编辑的菜单状态不会被保存。">
                    </el-alert>
                </el-form-item>
            </el-form>
        </el-main>
    </el-container>
</el-container>
</template>

<script>
import axios from "axios"
export default {
    data() {
        return {
            OperationType: "1",
            OperationMenuId: [],
            IsChildMenu: false,
            form: {
                MenuType: "Top",
                id: null,
                parentid: "0",
                title: "",
                number: "1",
                icon: "el-icon-location",
                linkto: "zongcheng",
                mode: "1"
            },
            MainMenu: []
        };
    },
    computed: {
        OperationSelect() {
            if (!this.MainMenu.length) {
                this.OperationType = "1";
            }
        },
        StringJson() {
            if (this.MainMenu.length) {
                return this.MainMenu;
            }
        },
        childMenu() {
            if (this.form.MenuType != "Top") {
                this.form.mode = "2";
            } else {
                this.form.mode = "1";
            }
        },
        parentMenu() {
            let parentMenuName = "请选择父菜单！";
            if (this.MainMenu.length && this.form.mode === "2") {
                try {
                    parentMenuName = this.MainMenu[this.form.parentid].title;
                    if (this.MainMenu[this.form.parentid].mode == 2) {
                        this.IsChildMenu = true;
                    }
                } catch {
                    let MenuArry = this.form.parentid.split("-");
                    let mainIndex = Number(MenuArry[0]);
                    let childIndex = Number(MenuArry[1]);
                    parentMenuName = this.MainMenu[mainIndex].children[childIndex].title;
                    this.IsChildMenu = true;
                }
            }
            return parentMenuName;
        }
    },
    methods: {
        LoadMenu() {
            let that = this
            this.$confirm("此操作将永久覆盖当前操作, 是否继续?", "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
            }).then(() => {
                axios.get("http://127.0.0.1:5000/adminmainmenu/").then(res => {

                    this.MainMenu = res.data.MainMenu
                    this.MesssageBox("success", "载入成功");
                }).catch(function (error) {
                    that.MesssageBox("error", error);
                })
            })

        },
        SaveMenu() {
            let that = this
this.$confirm("此操作将永久覆盖当前操作, 是否继续?", "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
            }).then(() => {
            axios.post("http://127.0.0.1:5000/adminmainmenu/", {
                "mainMenu": {
                    "MainMenu": this.MainMenu
                }
            }).then(res => {
                this.MesssageBox("success", "保存成功");
            }).catch(function (error) {
                that.MesssageBox("error", error);
            })
            })
        },
        addMenu() {
            if (this.MainMenu.length === 0 && this.form.MenuType === "Child") {
                this.alertMessage("<center>请先创建一个顶级菜单！</center>");
            } else if (this.form.title === "") {
                this.alertMessage("<center>请输入菜单名称！</center>");
            } else if (this.form.MenuType === "Child" && this.IsChildMenu) {
                this.alertMessage(
                    "<center>当前菜单不能添加子菜单。<p><h4>请选择“顶级导航类菜单“</h4></p></center>"
                );
            } else {
                let MenuJson;
                if (this.form.mode === "1") {
                    MenuJson = {
                        MenuType: this.form.MenuType,
                        title: this.form.title,
                        number: this.form.number,
                        icon: this.form.icon,
                        mode: this.form.mode,
                        children: []
                    };
                    this.MainMenu.push(MenuJson);
                    this.MesssageBox("success", "操作成功！");
                } else {
                    MenuJson = {
                        MenuType: this.form.MenuType,
                        parentid: this.form.parentid,
                        title: this.form.title,
                        number: this.form.number,
                        icon: this.form.icon,
                        linkto: this.form.linkto,
                        mode: this.form.mode
                    };
                    if (this.form.MenuType != "Top") {
                        let child = this.MainMenu[this.form.parentid].children.length;
                        this.MainMenu[this.form.parentid].children[child] = MenuJson;
                        this.MesssageBox("success", "操作成功！");
                    } else {
                        this.MainMenu.push(MenuJson);
                        this.MesssageBox("success", "操作成功！");
                    }
                }
                this.MainMenu.push();

                // console.log(this.MainMenu)
            }
        },
        modifyMenu() {
            if (!this.OperationMenuId.length) {
                this.alertMessage("<center>请选择要修改的菜单！</center>");
            } else if (this.OperationMenuId.length === 1) {
                this.MainMenu[this.OperationMenuId[0]].title = this.form.title;
                this.MainMenu[this.OperationMenuId[0]].icon = this.form.icon;
                this.MainMenu[this.OperationMenuId[0]].number = this.form.number;
                this.OperationMenuId = [];
                this.MesssageBox("success", "操作成功！");
                // console.log(this.MainMenu)
            } else if (this.OperationMenuId.length === 2) {
                this.MainMenu[this.OperationMenuId[0]].children[
                    this.OperationMenuId[1]
                ].title = this.form.title;
                this.MainMenu[this.OperationMenuId[0]].children[
                    this.OperationMenuId[1]
                ].icon = this.form.icon;
                this.MainMenu[this.OperationMenuId[0]].children[
                    this.OperationMenuId[1]
                ].number = this.form.number;
                this.OperationMenuId = [];
                this.MesssageBox("success", "操作成功！");
            }

            this.MainMenu.push();
        },
        deleteMenu() {
            if (!this.OperationMenuId.length) {
                this.alertMessage("<center>请选择要删除的菜单！</center>");
            } else {
                this.$confirm("此操作将永久删除该菜单及其子菜单, 是否继续?", "提示", {
                        confirmButtonText: "确定",
                        cancelButtonText: "取消",
                        type: "warning"
                    })
                    .then(() => {
                        console.log(this.OperationMenuId.length);
                        if (this.OperationMenuId.length === 1) {
                            // 删除元素的值
                            this.MainMenu.splice(this.OperationMenuId[0], 1);
                        } else if (this.OperationMenuId.length === 2) {
                            // 删除元素的值
                            this.MainMenu[this.OperationMenuId[0]].children.splice(
                                this.OperationMenuId[1],
                                1
                            );
                        }
                        this.OperationMenuId = [];
                        this.MainMenu.push();
                        this.MesssageBox("success", "删除成功");
                    })
                    .catch(() => {
                        this.MesssageBox("info", "已取消删除");
                    });

                // console.log(this.MainMenu)
            }
        },
        //MenuList菜单JSON,MenuId菜单ID
        GetMenu(MenuList, MenuId) {
            let newMenu;

            let MenuArry = MenuId.split("-");
            if (MenuArry.length > 1) {
                let mainIndex = Number(MenuArry[0]);
                let childIndex = Number(MenuArry[1]);
                this.OperationMenuId = [mainIndex, childIndex];
                newMenu = MenuList[mainIndex].children[childIndex];
                this.form.parentid = newMenu.parentid;
            } else {
                let ID = Number(MenuId);
                this.OperationMenuId = [ID];
                newMenu = MenuList[ID];
            }

            this.form.MenuType = newMenu.MenuType;
            this.form.title = newMenu.title;
            this.form.number = newMenu.number;
            this.form.icon = newMenu.icon;
            this.form.linkto = newMenu.linkto;
            this.form.mode = newMenu.mode;
        },

        MesssageBox(type = "info", message) {
            this.$message({
                type: type,
                message: message
            });
        },
        alertMessage(message) {
            this.$alert(message, "操作提示", {
                dangerouslyUseHTMLString: true
            });
        },
        handleSelece(key, keyPath) {
            if (this.OperationType != "1") {
                this.GetMenu(this.MainMenu, key);
            } else {
                this.form.parentid = key;
                this.IsChildMenu = false;
            }
            // console.log(key, keyPath);
        },
        handleOpen(key, keyPath) {
            this.form.parentid = key;
            this.IsChildMenu = false;
            if (this.OperationType != "1") {
                this.GetMenu(this.MainMenu, key);
            }

            // console.log(this.form.parentid, key, keyPath);
        },
        handleClose(key, keyPath) {
            this.form.parentid = key;
            this.IsChildMenu = false;
            if (this.OperationType != "1") {
                this.GetMenu(this.MainMenu, key);
            }
            // console.log(key, keyPath);
        }
    }
};
</script>
<style>
#mainMenues
{
    font-size: 40px;
}
</style>
