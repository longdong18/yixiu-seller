<template>
  <div class="info">
		<item-header :name="infoName" v-on:backParent="backParent" />
		
		<div class="info__name">
			<p>系统推荐分类</p>
			<cube-select
				v-model="category.name"
				:options="categoryName"
				@change="typeChange"
			/>
		</div>
		<van-field
			v-model="category.name"
			label="分类名称"
			placeholder="请输入分类名称"
		/>

		<!-- <van-field
			v-model="category.cover"
			label="封面图片链接"
			placeholder="请输入分类封面图片信息"
		/> -->

		<van-field
			v-model="category.name"
			label="分类描述"
			placeholder="请输入分类描述"
		/>

		<!-- <van-field
			v-model="category.parent"
			label="此分类的父级"
			placeholder="请输入次分类的父级"
		/> -->

		<van-button size="large" @click="submit">确认添加</van-button>

	</div>
</template>

<script>
import { Field, Button } from 'vant'
import ItemHeader from '../components/itemHeader'
export default {
	props: {
		parentCategory: String
	},
	components: {
		[Field.name]: Field,
		[Button.name]: Button,
		ItemHeader
	},
	data () {
		return {
			categoryList: [],
			categoryName: [],
			infoName: '添加手机分类',
			category: {
				type: sessionStorage.getItem('category'),
				name: '',
				cover: '',
				desc: '',
				// shop: '5ac83157bcbe58709c9bd47a',
				shop: JSON.parse(sessionStorage.getItem('shopData'))._id
			}
		}
	},
	async created() {
		let res = await this.$api.getData('https://m.yixiutech.com/category/phoneRepair');
		if(res.code == 200){
			this.categoryList = res.data;
			this.categoryName = res.data.map( item => {
				return item.name;
			})
		}
	},
	methods: {
		backParent () {
			this.$emit('backParent', true);
		},
		typeChange (value ,index) {

		},
		async submit () {
			if (this.category.name == '') {
				return ;
			}
			this.parentCategory != '' ? this.category.parent = this.parentCategory : null;
			const toast = this.$createToast({
				txt: '加载中...',
				type: 'loading'
			})
			let type = sessionStorage.getItem('category');
			window.back = true;
			this.category.type = type;
			toast.show();
			let categoryRes = await this.$api.sendData('https://m.yixiutech.com/category', this.category);
			toast.hide();
			if (categoryRes.code !== 200) {
				this.prompt(categoryRes.errMsg, 'error').show();
				return;	
			}
			this.prompt('提交成功!', 'correct').show();
			// 初始化
			this.category.name = '';
			this.$emit('updateCategory', true);

		}
	}
}
</script>

<style scoped>
.info {
	position: relative;
}

.info__name {
	display: flex;
	justify-content: flex-start;
	align-items: center;
	padding: 15px 15px;
	font-size: 14px;
}

.info__name p {
	width: 90px;
}

.van-button {
	margin-top: 50px;
}

.info__name .cube-select {
	width: 60%;
}
</style>
