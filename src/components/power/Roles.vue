<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>角色管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addRolesDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 角色列表区域 -->
            <el-table :data="rolelist" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row
                            :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
                            v-for="(item1, i1) in scope.row.children"
                            :key="item1.id"
                        >
                            <!-- 渲染一级权限 -->
                            <el-col :span="5">
                                <el-tag
                                    closable
                                    @close="removeRightById(scope.row, item1.id)"
                                >{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级和三级权限 -->
                            <el-col :span="19">
                                <!-- 通过for循环 嵌套渲染二级权限 -->
                                <el-row
                                    :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                                    v-for="(item2, i2) in item1.children"
                                    :key="item2.id"
                                >
                                    <!-- 二级权限 -->
                                    <el-col :span="6">
                                        <el-tag
                                            closable
                                            @close="removeRightById(scope.row, item2.id)"
                                            type="success"
                                        >{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <!-- 三级权限 -->
                                    <el-col :class="18">
                                        <el-tag
                                            type="warning"
                                            v-for="(item3) in item2.children"
                                            :key="item3.id"
                                            closable
                                            @close="removeRightById(scope.row, item3.id)"
                                        >{{item3.authName}}</el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <!-- 索引列 -->
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditRoleDialog(scope.row.id)">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click='removeRoleById(scope.row.id)'>删除</el-button>
                        <el-button
                            type="warning"
                            icon="el-icon-setting"
                            size="mini"
                            @click="showSetRightDialog(scope.row)"
                        >分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <!-- 分配权限的对话框 -->
        <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisible"
            width="50%"
            @close="setRightDialogClosed"
        >
            <!-- 树形控件 -->
            <el-tree
                :data="rightslist"
                :props="treeProps"
                show-checkbox
                node-key="id"
                default-expand-all
                :default-checked-keys="defKeys"
                ref="treeRef"
            ></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 添加角色对话框 -->
        <el-dialog
            title="添加角色"
            :visible.sync="addRolesDialogVisible"
            width="50%"
            @close="addRolesDialogClosed"
        >
            <!-- 内容主体区 -->
            <el-form :model="addRolesForm" ref="addRolesFormRef" label-width="70px">
                <el-form-item label="角色名" prop="roleName">
                    <el-input v-model="addRolesForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc">
                    <el-input v-model="addRolesForm.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部按钮区 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addRolesDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addRole">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 修改角色的对话框 -->
            <el-dialog
                title="修改角色"
                :visible.sync="editRoleDialogVisible"
                width="50%"
                @close="editRoleDialogClosed"
            >
                <el-form
                    :model="editRoleForm"
                    ref="editRoleFormRef"
                    label-width="70px"
                >
                    <el-form-item label="角色名" prop="roleName">
                        <el-input v-model="editRoleForm.roleName" ></el-input>
                    </el-form-item>
                    <el-form-item label="角色描述" prop="roleDesc">
                        <el-input v-model="editRoleForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="editRoleDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="editRoleInfo">确 定</el-button>
                </span>
            </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            // 所有角色列表数据
            rolelist: [],
            // 控制显示权限对话框的显示与隐藏
            setRightDialogVisible: false,
            // 所有权限的数据
            rightslist: [],
            // 树形控件的属性绑定对象
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            // 默认选中的节点id值数组
            defKeys: [],
            // 即将分配角色的权限id
            roleId: '',
            // 控制添加角色对话框的显示与隐藏
            addRolesDialogVisible: false,
            // 添加角色的表单数据
            addRolesForm: {
                roleName: '',
                roleDesc: ''
            },
            // 查询到的角色信息对象
            editRoleForm: {},
            // 控制修改绝对的对话框的显示与隐藏
            editRoleDialogVisible: false
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        // 获取所有角色的列表
        async getRolesList() {
            const { data: res } = await this.$http.get('roles')
            if (res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败')
            }
            this.rolelist = res.data
            // console.log(this.rolelist)
        },
        // 根据id删除对应的权限
        async removeRightById(role, rightId) {
            // 弹框提示用户是否删除
            const confirmResult = await this.$confirm(
                '此操作将永久删除该文件, 是否继续?',
                '提示',
                {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }
            ).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('取消了删除')
            }
            const { data: res } = await this.$http.delete(
                `roles/${role.id}/rights/${rightId}`
            )
            if (res.meta.status !== 200) {
                return this.$message.error('删除权限失败')
            }
            role.children = res.data
        },
        // 展示分配权限的对话框
        async showSetRightDialog(role) {
            this.roleId = role.id
            // 获取所有权限数据
            const { data: res } = await this.$http.get('rights/tree')
            if (res.meta.status !== 200) {
                return this.$message.error('获取权限数据失败')
            }
            // 把获取到的权限数据保存到 data 中
            this.rightslist = res.data
            // console.log(this.rightslist)
            // 递归获取三级节点的id
            this.getLeafKeys(role, this.defKeys)
            // 展示树形结构
            this.setRightDialogVisible = true
        },
        // 通过递归的形式，获取角色下所有三级权限的id，并保存到defKeys数组中
        getLeafKeys(node, arr) {
            // 如果当前node节点不包括children属性，则是三级节点
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item => {
                this.getLeafKeys(item, arr)
            })
        },
        // 监听分配权限对话框的关闭事件
        setRightDialogClosed() {
            this.defKeys = []
        },
        // 点击为角色分配权限
        async allotRights() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',')
            const { data: res } = await this.$http.post(
                `roles/${this.roleId}/rights`,
                { rids: idStr }
            )
            if (res.meta.status !== 200) {
                return this.$message.error('分配权限失败')
            }
            this.$message.success('分配权限成功')
            this.getRolesList()
            this.setRightDialogVisible = false
        },
        // 点击按钮，添加角色
        addRole() {
            this.$refs.addRolesFormRef.validate(async valid => {
                // console.log(valid)
                if (!valid) return
                // 可以发起添加用户的网络请求
                const { data: res } = await this.$http.post(
                    'roles',
                    this.addRolesForm
                )
                if (res.meta.status !== 201) {
                    this.$message.error('添加角色失败')
                }
                this.$message.success('添加角色成功')
                // 隐藏添加角色的对话框
                this.addRolesDialogVisible = false
                // 重新获取角色列表数据
                this.getRolesList()
            })
        },
        // 监听添加角色对话框关闭事件
        addRolesDialogClosed() {
            this.$refs.addRolesFormRef.resetFields()
        },
        // 展示编辑角色的对话框
        async showEditRoleDialog(id) {
            const { data: res } = await this.$http.get('roles/' + id)
            // console.log(res)
            if (res.meta.status !== 200) {
                return this.$message.error('修改角色信息失败')
            }
            this.editRoleForm = res.data
            this.editRoleDialogVisible = true
        },
        // 监听修改角色对话框关闭事件
        editRoleDialogClosed() {
            this.$refs.editRoleFormRef.resetFields()
        },
        // 修改用户信息并提交
        editRoleInfo() {
            this.$refs.editRoleFormRef.validate(async valid => {
                // console.log(valid)
                if (!valid) return
                // 发起修改用户的请求,注意这里的id是角色id
                // 【注意】关于角色名可以改，但是角色描述没有修改成功的问题:这里虽然有两个属性，但是只能解构一个大括号，写成两个大括号的内容，解构两次，虽然不会报错，但只能修改第一个括号里的内容，而角色名又是必须要获取到解构出来的内容，故，两个属性之间用逗号隔开
                const { data: res } = await this.$http.put('roles/' + this.editRoleForm.roleId, { roleName: this.editRoleForm.roleName, roleDesc: this.editRoleForm.roleDesc })
                // console.log(res)
                if (res.meta.status !== 200) {
                    return this.$message.error('更新角色信息失败')
                }
                // 关闭对话框
                this.editRoleDialogVisible = false
                // 刷新数据列表
                this.getRolesList()
                // 提示修改成功
                this.$message.success('更新角色信息成功')
            })
        },
        // 根据id删除对应的角色
        async removeRoleById(id) {
            // console.log(id)
            // 弹框进行提示用户是否删除数据
            const confirmResult = await this.$confirm(
                '此操作将永久删除该角色, 是否继续?',
                '提示',
                {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }
            ).catch(err => {
                return err
            })
            // 如果用户确认删除，则返回值为字符串 confirm
            // 如果用户取消了删除，则返回值为字符串 cancel
            // console.log(confirmResult)
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            // console.log('确认了删除')
            const { data: res } = await this.$http.delete('roles/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error('删除角色失败')
            }
            this.$message.success('删除角色成功')
            this.getRolesList()
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px;
}
.bdtop {
    border-top: 1px solid #eee;
}
.bdbottom {
    border-bottom: 1px solid #eee;
}
.vcenter {
    display: flex;
    align-items: center;
}
</style>
