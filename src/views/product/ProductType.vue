<template id="productType">
	<el-row style="height: 100%;border: 1px solid rebeccapurple;margin-top: 10px">
		<el-col :span="6" style="border-right: 1px solid rebeccapurple">
			<div class="grid-content bg-purple">
				<el-tree :data="productTypes" :props="defaultProps" @node-click="handleNodeClick">

				</el-tree>
			</div>
		</el-col>
		<el-col :span="17" style="margin-left: 10px;padding-top: 10px">
			<div>
				<el-form>
					<el-form-item label="活动名称">
						<el-input style="width: 200px"></el-input>
					</el-form-item>
					<el-form-item label="活动名称">
						<el-input style="width: 200px"></el-input>
					</el-form-item>
					<el-form-item label="活动名称">
						<el-input style="width: 200px"></el-input>
					</el-form-item>
					<el-form-item label="活动名称">
						<el-input style="width: 200px"></el-input>
					</el-form-item>
					<el-form-item label="活动名称">
						<el-input style="width: 200px"></el-input>
					</el-form-item>
					<el-form-item>
						<el-button type="primary" @click="submitForm('ruleForm')">提交</el-button>
						<el-button @click="resetForm('ruleForm')">重置</el-button>
					</el-form-item>
				</el-form>
			</div>

		</el-col>
	</el-row>
</template>
<style>
	.el-row {
		margin-bottom: 20px;
		height: 100%;
		max-height: 100%;
	}
	:last-child {
		margin-bottom: 0;
	}
	#productType el-col {
		border: 1px solid red;
		border-radius: 4px;
	}
	.grid-content {
		border-radius: 4px;
		min-height: 36px;
	}
</style>

<script>
    export default {
        data() {
            return {
                form: {
                    name: '',
                    region: '',
                    date1: '',
                    date2: '',
                    delivery: false,
                    type: [],
                    resource: '',
                    desc: ''
                },
                productTypes:[],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                }
            }
        },
        methods:{
            handleNodeClick(data) {
                console.log(data);
            },
			loadProductType(){
                this.$http.get("/product/productType/list")
					.then(res=>{
					    let data = res.data;
					    this.productTypes = data;
					})
			}
        },
        mounted(){
			this.loadProductType();
        }
    };
</script>