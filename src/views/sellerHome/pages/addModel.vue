<template>
  <div class="info">

		<item-header :name="infoName" v-on:backParent="backParent" />

		<van-field
			v-model="brandName"
			label="品牌名称"
			placeholder="品牌名称"
			disabled
		/>

		<div class="info__name">
			<p>系统推荐型号</p>
			<cube-select
				v-model="model.name"
				:options="phoneModel"
				@change="modelChange"
			/>
		</div>

		<!-- <van-field
			v-model="model.name"
			label="其他型号"
			placeholder="如果没有你想要的型号名,请填写你想要的型号名"
		/> -->

		<!-- <van-field
			v-model="model.desc"
			label="描述"
			placeholder="请输入品牌描述"
		/> -->

		<!-- <van-field
			v-model="model.cover"
			label="封面"
			placeholder="请输入封面地址"
		/> -->

		<!-- <div class="info__name">
			<p>手机颜色</p>
		</div> -->

		<!-- <div class="service">
			<selects ref="select" v-for="(item, index) in phoneModelColor"
				:key="index"
				:data="item"
				v-on:sendMsg="sendMsg"
				v-on:remove="remove"
			/>
		</div> -->

		<!-- <van-field
			v-model="otherColors"
			label="其他颜色"
			placeholder="请输入其他颜色,并以中文逗号相隔"
		/> -->

		<van-button size="large" @click="submit">确认添加</van-button>

	</div>
</template>

<script>
import { Field, Button } from 'vant'
import ItemHeader from '../components/itemHeader'
import selects from '../components/select'
export default {
	props: {
		brandName: String,
		brandId: String,
		sysModel: Array,
		phoneModel: Array,
		manufacturer: String,
		modelNames: Array
	},
  components: {
		[Field.name]: Field,
		[Button.name]: Button,
		ItemHeader,
		selects
	},
	async mounted () {
		const toast = this.$createToast({
			txt: '加载中...',
			type: 'loading'
		})
		toast.show();
		
		let res = await this.$api.getData('https://m.yixiutech.com/phone/manufacturer/shop/' + this.model.shop);
		toast.hide();
		res.data.map(item => {
			this.phoneName.push(item.name);
			this.phoneInfo.push(item);
		})
		// let own
	},
	data () {
		return {
			infoName: '添加手机型号',
			colors: ['黑色', '白色', '银灰色'],
			otherColors: [],
			phoneName: [],
			phoneInfo: [],
			// phoneModel: [],
			phoneModelAlias: '',
			phoneModelInfo: [],
			phoneModelColor: [],
			phoneModelColorRes: [],
			colorType: 'color',
			model: {
				name: '',
				// shop: '5ac83157bcbe58709c9bd47a',
				shop: JSON.parse(sessionStorage.getItem('shopData'))._id,
				color: [],
				manufacturer: ''
			}
		}
	},
	methods: {
		sendMsg (data) {
			this.model.color.push(data);
		},
		remove(data) {
			this.model.color.map( (item, index) => {
				item == data ? this.model.color.splice(index, 1) : null
			})
		},
		backParent () {
			this.phoneModelColor = [];
			this.model.name = '';
			this.$emit('backParent', true);
		},
		modelChange (value, index, text) {
			this.modelName = text;
			this.sysModel.map(item => {
				item._id == value ? this.model.color = item.color : null;
			})

			// this.model['name'] = value;
			// this.phoneModelColor = this.phoneModelInfo[ index ].color;
			
		},
		async submit () {
			let status = true;
			this.modelNames.map(item => {
				if (item == this.modelName) {
					this.prompt('店铺里已添加过同名的型号', 'error').show();
					return;
				}
			})

			this.model.name = this.modelName;
			this.model.manufacturer = this.manufacturer;

			if (!this.model.name) {
				alert('请选择需要添加的手机型号');
				return;
			}

			if (status) {
				const toast = this.$createToast({
					txt: '加载中...',
					type: 'loading'
				})
				toast.show();
				let modelRes = await this.$api.sendData('https://m.yixiutech.com/phone/model', this.model);
				toast.hide();
				if (modelRes.code !== 200) {
					this.prompt(modelRes.errMsg, 'error').show();
					return;	
				}
				this.prompt('添加成功!', 'correct').show();
				this.model = Object.assign(this.model, {color: [], manufacturer: '', alias: '', name: ''})
				this.$emit('updateModel', true);
			}
		}
	}
}
</script>

<style scoped>
.info {
	background: #fff;
}
.info__name {
	display: flex;
	justify-content: flex-start;
	align-items: center;
	padding: 0 15px;
	margin: 10px 0;
	font-size: 14px;
}

.info__name p {
	width: 90px;
}

.van-button {
	margin-top: 50px;
}

.service {
	width: 92%;
	display: flex;
	flex-wrap: wrap;
	justify-content:space-start;
	padding: 4%;
}

.info__name .cube-select {
	width: 60%;
}
</style>
