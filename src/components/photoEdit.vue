<template>
	<div class="edit">
		<h1>Photo Edit</h1>

		<!-- 流程条 -->
		<el-steps :space="200" :active="active" finish-status="success">
			<el-step :title="imgUrl === undefined ? '待上传' : '已上传'"></el-step>
			<el-step title="待编辑"></el-step>
			<el-step title="待下载"></el-step>
		</el-steps>

		<!-- 上传文件按钮 -->
		<input
			class="uploadBtn"
			type="file"
			accept="image/*"
			@change="handleFileChange"
		/>

		<!-- 缺省动画 -->
		<el-skeleton v-if="imageUrl === ''" :rows="6" animated />

		<!-- 日期、地点输入框 -->
		<div class="editBox" v-if="imageUrl !== ''">
			<div class="inputBox">
				时间：
				<el-time-select
					v-model="timeValue"
					:picker-options="{
						start: '07:55',
						step: '00:01',
						end: '08:05',
					}"
					placeholder="选择时间"
				>
				</el-time-select>
			</div>
			<div class="inputBox">
				日期：
				<el-date-picker
					v-model="dateValue"
					type="date"
					value-format="yyyy-MM-dd"
					placeholder="选择日期"
				>
				</el-date-picker>
			</div>
			<div class="inputBox">
				周几:
				<el-select v-model="weekValue" placeholder="请选择">
					<el-option
						v-for="item in weekOptions"
						:key="item.value"
						:label="item.label"
						:value="item.value"
					>
					</el-option>
				</el-select>
			</div>
			<div class="inputBox">
				地点:
				<el-input class="addInput" v-model="add" placeholder="请输入" />
			</div>

			<div>
				<el-button class="saveBtn" type="primary" @click="saveToImage"
					>保存图片</el-button
				>
			</div>
		</div>

		<!-- 图片预览区域 -->
		<div
			v-if="imageUrl !== ''"
			ref="vueDomSaveToImage"
			:class="imgDirection === '1' ? 'imgBox1' : 'imgBox0'"
		>
			<div :class="imgDirection === '1' ? 'imgCut0' : 'imgCut1'">
				<img class="img" :src="imageUrl" alt="" />
			</div>

			<div class="time">{{ timeValue }}</div>
			<div class="details">
				<span class="date">{{ dateValue }}</span>
				<div class="empty"></div>
				{{ '星期' + weekValue + '' }}
				<div class="empty"></div>
				<img src="@/assets/ding.png" alt="" class="ding" />
				{{ add }}
			</div>

			<img src="../assets/logo.png" class="logo" alt="" />
		</div>
	</div>
</template>

<script>
import html2canvas from 'html2canvas';

export default {
	name: 'photoEdit',
	components: {},
	data() {
		return {
			active: 1, //当前进度
			imageUrl: '', //图片地址
			fileName: '',
			add: ' 西安市·汇成·和苑',
			timeValue: '07:58',
			dateValue: '2023-08-25',
			weekValue: '五',
			weekOptions: [
				{ label: '一', value: '一' },
				{ label: '二', value: '二' },
				{ label: '三', value: '三' },
				{ label: '四', value: '四' },
				{ label: '五', value: '五' },
				{ label: '六', value: '六' },
				{ label: '日', value: '日' },
			], //周 下拉框数据
			imgDirection: '0', //0：0：横版 1：竖向
		};
	},
	mounted() {},

	methods: {
		handleFileChange(e) {
			const file = e.target.files[0];
			// 取上传文件的文件名
			this.fileName = file.name.substring(0, file.name.lastIndexOf('.'));
			const fileReader = new FileReader();
			fileReader.onload = (e) => {
				const img = new Image();
				img.onload = () => {
					const width = img.width;
					const height = img.height;
					const ratio = width / height;
					if (ratio > 0.9) {
						console.log('横向图片');
						this.imgDirection = '0';
					} else {
						console.log('竖向图片');
						this.imgDirection = '1';
					}
				};
				img.src = e.target.result;
				this.imageUrl = img.src;

				const timeStr = '07:58';
				const [hours, minutes] = timeStr.split(':');
				let houra = hours;

				const offset = Math.floor(Math.random() * 11 - 1);

				let newMinutes = Number(minutes) + offset;
				if (newMinutes >= 60) {
					houra++;
					newMinutes -= 60;
				} else if (newMinutes < 0) {
					houra--;
					newMinutes += 60;
				}

				this.timeValue = `${houra.toString().padStart(2, '0')}:${newMinutes
					.toString()
					.padStart(2, '0')}`;
				this.active = 2;
			};
			fileReader.readAsDataURL(file);
		},
		saveToImage() {
			this.$nextTick(() => {
				html2canvas(this.$refs.vueDomSaveToImage).then((res) => {
					console.log(res, 'res');
					let imgUrl = res.toDataURL('image/png');
					const save_link = document.createElementNS(
						'http://www.w3.org/1999/xhtml',
						'a'
					);
					save_link.href = imgUrl;
					save_link.download = this.fileName + '.png'; //保存文件名格式为 ***-点名册、会议纪要、等文件名
					const event = document.createEvent('MouseEvents');
					event.initMouseEvent(
						'click',
						true,
						false,
						window,
						0,
						0,
						0,
						0,
						0,
						false,
						false,
						false,
						false,
						0,
						null
					);
					save_link.dispatchEvent(event);
					this.imageUrl = '';
					this.fileName = '';
					this.active = 1; //步骤置为1
					this.$notify({
						title: '成功',
						message: '保存成功!',
						type: 'success',
					});
					this.$forceUpdate();
				});
			});
		},
	},
};
</script>

<style scoped>
.edit {
	padding: 0 100px;
}

.uploadBtn {
	width: 500px;
	height: 60px;
}
.editBox {
	display: flex;
	font-size: 26px;
	font-weight: 500;
	margin: 0px 0 50px 0;
}
.inputBox {
	margin-right: 50px;
}
.input {
	width: 200px;
	height: 40px;
	border: rgb(22, 169, 238) 3px solid;
	border-radius: 8px;
}
.addInput {
	width: 180px;
}
/* 横向 */
.imgBox0 {
	width: 1240px;
	height: 720px;
	position: relative;
	background: #c4bebe;
	color: #fff;
}
/* 竖向 */
.imgBox1 {
	width: 720px;
	height: 1240px;
	position: relative;
	background: #c4bebe;
	color: #fff;
}
/* 横向 */
.imgCut1 {
	width: 1240px;
	height: 720px;
	/* position: relative; */
}
/* 竖向 */
.imgCut0 {
	width: 720px;
	height: 1240px;
	/* position: relative; */
}
.img {
	width: 100%;
	height: 100%;
	object-fit: cover;
	/* position: absolute; */
}
.date {
	font-size: 32px;
}
.ding {
	width: 40px;
}
.empty {
	width: 10px;
}
.time {
	width: 300px;
	position: absolute;
	bottom: 13%;
	left: 50%;
	margin-left: -150px;
	font-size: 125px;
	font-weight: 400;
	letter-spacing: 2px;
	text-shadow: 3px 3px 3px #807f7f;
}
.details {
	width: 600px;
	position: absolute;
	left: 50%;
	margin-left: -320px;
	bottom: 9%;
	font-size: 30px;
	display: flex;
	justify-content: center;
	text-shadow: 3px 3px 3px #807f7f;
}

.logo {
	position: absolute;
	height: 50px;
	right: 10px;
	bottom: 10px;
}
</style>
