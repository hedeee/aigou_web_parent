<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="名称"></el-input>
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
			<el-table-column prop="name" label="名称" width="180" sortable>
			</el-table-column>
			<el-table-column prop="englishName" label="英文名" width="180"  sortable>
			</el-table-column>
			<el-table-column prop="productTypeId" label="类型" width="180" sortable>
			</el-table-column>
			<el-table-column prop="description" label="描述" width="270" sortable>
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
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="10" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="功能操作" v-model="formVisible" :close-on-click-modal="false">
			<el-form :model="form" label-width="80px" :rules="formRules" ref="form">
				<el-form-item label="名称" prop="name">
					<el-input v-model="form.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="英文名" prop="englishName">
					<el-input v-model="form.englishName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="logo" prop="logo">
					<el-upload
							class="upload-demo"
							action="http://127.0.0.1:9527/services/common/upload"
							:on-preview="handlePreview"
							:on-remove="handleRemove"
							:file-list="fileList2"
							:on-success="handleSuccess"
							list-type="picture">
						<el-button size="small" type="primary">点击上传</el-button>
						<div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
					</el-upload>
				</el-form-item>
				<el-form-item label="排序号" prop="sortIndex">
					<el-input-number v-model="form.sortIndex" auto-complete="off"></el-input-number>
				</el-form-item>
				<el-form-item label="类型" prop="productTypeId">
					<el-input v-model="form.productTypeId" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="描述" prop="description">
					<el-input type="textarea" v-model="form.description"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="formVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>

	export default {
		data() {
			return {
				filters: {
					keyword: ''
				},
				brands: [],
				total: 0,
				page: 1,
				listLoading: false,
				sels: [],//列表选中列
                fileList2: [],
				formVisible: false,//编辑界面是否显示
				editLoading: false,
				formRules: {
					name: [
						{ required: true, message: '请输入名称', trigger: 'blur' }
					]
				},
				//编辑界面数据
                form: {
                    name: '',
                    englishName: '',
                    sortIndex: '',
                    description: '',
                    logo: '',
                    productTypeId: 0
                },
			}
		},
		methods: {
            //移除logo后
		    handleRemove(file, fileList) {
                console.log(file, fileList)
            //    移除后file中也有response值与上传时候一样，下面传入这个logo的路径
                var filePath=file.response.resultObj;
				this.$http.delete("/common/del?filePath="+filePath)
					.then(res=>{
					    if(res.data.success){
                            this.$message({
                                message: '删除成功',
                                type: 'success'
                            });
						}else {
                            this.$message({
                                message: '删除失败',
                                type: 'error'
                            });
						}
					})
            },
            handlePreview(file) {
                console.log(file);
            },
			//上传成功后执行的
            handleSuccess(response, file, fileList) {
				console.log(response, file, fileList)
            	//上传一个文件测试，response响应的值为下面,file中也有response
				// message: "操作成功"
            	//resultObj: "/group1/M00/00/01/rBAOTlxAfoKAML6NAABrl6r7ysY382.JPG"
            	//success: true
            	//所以直接将log的值与object等起来即可
                this.form.logo = file.response.resultObj;
            },

			handleCurrentChange(val) {
				this.page = val;
				this.getBrands();
			},
			//获取用户列表
			getBrands() {
				let para = {
					page: this.page,
					keyword: this.filters.keyword
				};
				this.listLoading = true;
				//NProgress.start();
				
				this.$http.post("/product/brand/json",para)
					.then((res) => {
					this.total = res.data.total;
					this.brands = res.data.rows;
					this.listLoading = false;
					//NProgress.done();
				});
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					this.$http.delete("/product/brand/"+row.id)
					.then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getBrands();
					});
				}).catch(() => {

				});
			},
			//显示编辑界面
			handleEdit: function (index, row) {
				this.formVisible = true;
				//回显
				this.form = Object.assign({}, row);
                //每次点击编辑因为fileList2已经装入了数据，所以会一直增多，要先清空后再回显
				this.fileList2=[];

                //回显缩略图
                this.fileList2.push({
					"url":"http://172.16.14.78"+row.logo
                })
			},
            //显示新增界面
            handleAdd: function () {
		        //与编辑的回显同理，只有初始化的时候没有数据，以后都会有数据，要先清空
                this.fileList2=[];
		        this.formVisible = true;

                this.form = {};
            },
			//编辑
			editSubmit: function () {
				this.$refs.form.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {})
							.then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.form);
							console.log(para);
							this.$http.post("/product/brand/save",para)
								.then((res) => {
								this.editLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['form'].resetFields();
								this.formVisible = false;
								this.getBrands();
							});
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
					batchRemoveUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getBrands();
					});
				}).catch(() => {

				});
			}
		},
		mounted() {
			this.getBrands();
		}
	}

</script>

<style scoped>

</style>