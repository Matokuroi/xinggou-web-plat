<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getBrands">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="brands" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="name" label="品牌" width="120" sortable>
			</el-table-column>
			<el-table-column prop="englishName" label="英文名" width="150"  sortable>
			</el-table-column>
			<el-table-column prop="logo" label="品牌LOGO" width="180" sortable>
				<template scope="scope">
					<img :src="'http://192.168.1.106/'+scope.row.logo" height="50"/>
				</template>
			</el-table-column>
			<el-table-column prop="productType.name" label="商品类型" width="180" sortable>
			</el-table-column>
			<el-table-column prop="description" label="描述" min-width="180" sortable>
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="编辑" :visible.sync="editFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="品牌名称" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="英文名称" prop="englishName">
					<el-input v-model="editForm.englishName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="类型" prop="productTypeId">
					<el-input v-model="editForm.productTypeId" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="描述">
					<el-input type="textarea" v-model="editForm.description"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="editFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--新增界面-->
		<el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false" size="tiny">
			<el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
				<el-form-item label="品牌名称" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="英文名称" prop="name">
					<el-input v-model="addForm.englishName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="类型" prop="productTypeId">
					<!--<el-input v-model="addForm.productTypeId" auto-complete="off"></el-input>-->
					<el-select :value="valueTitle" :clearable="clearable" @clear="clearHandle">
					<el-option :value="valueTitle" :label="valueTitle">
						<el-tree  id="tree-option"
								  ref="selectTree"
								  :accordion="accordion"
								  :data="productTypes"
								  :props="props"
								  :node-key="props.value"
								  :default-expanded-keys="defaultExpandedKey"
								  @node-click="handleNodeClick">
						</el-tree>
					</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="logo" prop="logo">
					<el-upload
							class="upload-demo"
							action="http://127.0.0.1:2000/services/common/upload"
							:on-preview="handlePreview"
							:on-remove="handleRemove"
							:file-list="fileList2"
							:multiple="false"
							:on-success="handleSuccess"
							:before-upload="handleBeforeUpload"
							list-type="picture"
					>
						<el-button size="small" type="primary">点击上传</el-button>
						<div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
					</el-upload>
				</el-form-item>
				<el-form-item label="描述">
					<el-input type="textarea" v-model="addForm.description"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="cancel()">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

	export default {
        name: "el-tree-select",
        props:{
            /* 配置项 */
            props:{
                type: Object,
                default:()=>{
                    return {
                        value:'id',             // ID字段名
                        label: 'name',         // 显示名称
                        children: 'children'    // 子级字段名
                    }
                }
            },

            /* 初始值 */
            value:{
                type: Number,
                default: ()=>{ return null }
            },
            /* 可清空选项 */
            clearable:{
                type:Boolean,
                default:()=>{ return true }
            },
            /* 自动收起 */
            accordion:{
                type:Boolean,
                default:()=>{ return false }
            },
        },
		data() {
			return {
                fileList2:[],
                productTypes:[],
                valueId:this.value,    // 初始值
                valueTitle:'',
                defaultExpandedKey:[],

				filters: {
					keyword: ''
				},
                pageSize:10,
				brands: [],
				total: 0,
				page: 1,
				listLoading: false,
				sels: [],//列表选中列

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
					id: 0,
                    name: '',
                    englishName:'',
                    productTypeId:null,
                    description:'',
					logo:''
				},

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
                    name: '',
                    englishName:'',
                    productTypeId:null,
                    description:'',
                    logo:''
				}

			}
		},

		methods: {
            cancel(){
                	if (this.addForm.logo){

                        var filePath = this.addForm.logo;
                        console.log("qawseq"+filePath)
                        this.$http.delete("/common/del?filePath=" + filePath)
                            .then(res => {
                                if (res.data.success) {
                                    this.$message({
                                        message: '退出，清空fastdfs图片成功!',
                                        type: 'success'
                                    });

                                } else {
                                    this.$message({
                                        message: '退出，清空图片失败!',
                                        type: 'error'
                                    });
                                }

                            })
                    }

				//this.$refs.addForm.resetFields()

                this.addFormVisible = false;
			},
            //上传之前
            handleBeforeUpload(){
                console.debug(this.fileList2.length)
                if(this.fileList2.length>0){
                    this.$message({
                        message: '只能上传一个文件',
                        type: 'error'
                    });
                    return false;
                }
            },
            handleSuccess(response, file, fileList){
                //上传成功回调
                //console.log(response, file, fileList)
                console.log(this.addForm.logo)
                this.addForm.logo = file.response.resultObj;
                console.log(this.addForm.logo)
                this.fileList2 = fileList;

            },
            handleRemove(file, fileList) {
                if(file.response){
                    var filePath =file.response.resultObj;
                    this.$http.delete("/common/del?filePath="+filePath)
                        .then(res=>{
                            if(res.data.success){
                                this.$message({
                                    message: '删除成功!',
                                    type: 'success'
                                });
                                this.fileList2 = fileList;
                            }else{
                                this.$message({
                                    message: '删除失败!',
                                    type: 'error'
                                });
                            }
                        })
                }
            },
            handlePreview(file) {
                console.log(file);
            },
            // 初始化值
            initHandle(){
                if(this.valueId){
                    this.valueTitle = this.$refs.selectTree.getNode(this.valueId).data[this.props.label] ;    // 初始化显示
                    this.$refs.selectTree.setCurrentKey(this.valueId);  // 设置默认选中
                    this.defaultExpandedKey = [this.valueId]      // 设置默认展开
                }
				this.initScroll();
            },
			// 初始化滚动条
            initScroll(){
                this.$nextTick(()=>{
                    let scrollWrap = document.querySelectorAll('el-select-dropdown__wrap .el-scrollbar')
                    let scrollBar = document.querySelectorAll('.el-scrollbar .el-scrollbar__bar')
					console.log("scrollWrap",scrollWrap);
					console.log(scrollBar);
                    //scrollWrap.style.cssText = 'margin: 0px; max-height: none; overflow: hidden;'
                    scrollBar.forEach(ele => ele.style.width = 0)
                })
			},
            // 切换选项
            handleNodeClick(node){
                this.addForm.productTypeId=node[this.props.value]
                this.valueTitle = node[this.props.label]
                this.valueId = node[this.props.value]
                this.$emit('getValue',this.valueId)
                this.defaultExpandedKey = []
            },
            // 清除选中
            clearHandle(){
                this.valueTitle = ''
                this.valueId = null
                this.defaultExpandedKey = []
                this.clearSelected()
                this.$emit('getValue',null)
            },
            /* 清空选中样式 */
            clearSelected(){
                let allNode = document.querySelectorAll('#tree-option .el-tree-node')
                allNode.forEach((element)=>element.classList.remove('is-current'))
            },
            loadProductTypes(){
                this.$http.get("/product/productType/list")
                    .then(res=>{
                        let data = res.data;
                        this.productTypes = data;
                    })
            },
			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getBrands();
			},
			//获取用户列表
			getBrands() {
				this.listLoading = true;
				this.$http.post("/product/brand/json",{
				    pageNum:this.page,
					pageSize:this.pageSize,
					keyword:this.filters.keyword
				}).then(res=>{
                    this.listLoading = false;
                    let pageList = res.data;
                    this.brands = pageList.rows;
                    this.total = pageList.total;
				})
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();

					this.$http.delete("/product/brand/delete/"+row.id)
						.then(res=>{
                            this.listLoading = false;
                            if(res.data.success){
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                                this.getBrands();
                            }else {
                                this.$message({
                                    message: res.data,
                                    type: 'error'
                                });
							}
						})
					/*removeUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getBrands();
					});*/
				})
			},
			//显示编辑界面
			handleEdit: function (index, row) {
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
			},
			//显示新增界面
			handleAdd: function () {
				this.addFormVisible = true;
				this.addForm = {
                    name: '',
                    englishName:'',
                    productTypeId:null,
                    description:''
				};
			},
			//编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.editForm);
							/*para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							editUser(para).then((res) => {
								this.editLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['editForm'].resetFields();
								this.editFormVisible = false;
								this.getBrands();
							});*/
							this.$http.post("/product/brand/add",para)
								.then(res=>{
								    let data = res.data;
								    if (data.success){
                                        this.editLoading = false;
                                        //NProgress.done();
                                        this.$message({
                                            message: '提交成功',
                                            type: 'success'
                                        });
                                        this.$refs['editForm'].resetFields();
                                        this.editFormVisible = false;
                                        this.getBrands();
									}else {
                                        this.$message({
                                            message: data.message,
                                            type: 'error'
                                        });
									}
								})
						});
					}
				});
			},
			//新增
			addSubmit: function () {
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.addForm);
							this.$http.post("/product/brand/add",para)
								.then(res=>{
                                    this.addLoading = false;
                                    let data = res.data;
                                    //NProgress.done();
									if(data.success){
                                        this.$message({
                                            message: '提交成功',
                                            type: 'success'
                                        });
                                        this.$refs['addForm'].resetFields();
                                        this.addFormVisible = false;
                                        this.getBrands();
									}else {
                                        this.$message({
                                            message: data.message,
                                            type: 'error'
                                        });
									}

								})
							/*para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							addUser(para).then((res) => {
								this.addLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['addForm'].resetFields();
								this.addFormVisible = false;
								this.getBrands();
							});*/
						});
					}
				});
			},
			selsChange: function (sels) {
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { ids: ids };
                    this.$http.post("/product/brand/deleteMany",para)
						.then(res=>{
						    let data = res.data;
                            this.listLoading = false;
						    if(data.success){
                                //NProgress.done();
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                                this.getBrands();
							}else {
                                this.$message({
                                    message: data.message,
                                    type: 'error'
                                });
							}

						})
					/*batchRemoveUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getBrands();
					});*/
				})
			}
		},
        watch: {
            value(){
                this.valueId = this.value
                this.initHandle()
            }
        },
		mounted() {
				this.getBrands();
				this.loadProductTypes();
				this.initHandle();
			}
		}

</script>

<style scoped>
	.el-scrollbar .el-scrollbar__view .el-select-dropdown__item{
		height: auto;
		max-height: 274px;
		padding: 0;
		overflow: hidden;
		overflow-y: auto;
	}
	.el-select-dropdown__item.selected{
		font-weight: normal;
	}
	ul li >>>.el-tree .el-tree-node__content{
		height:auto;
		padding: 0 20px;
	}
	.el-tree-node__label{
		font-weight: normal;
	}
	.el-tree >>>.is-current .el-tree-node__label{
		color: #409EFF;
		font-weight: 700;
	}
	.el-tree >>>.is-current .el-tree-node__children .el-tree-node__label{
		color:#606266;
		font-weight: normal;
	}

</style>