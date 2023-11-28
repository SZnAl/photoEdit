<template>
	<div class="edit">
		<div class="title">
			<h1>Photo Edit V{{ version }}</h1>
			<el-tooltip
				class="item"
				effect="dark"
				content="点击查看提示"
				placement="right"
			>
				<em class="el-icon-info tipsIcon" @click="dialogVisible = true"></em>
			</el-tooltip>
		</div>

		<!-- 流程条 -->
		<el-steps :active="active" finish-status="success">
			<el-step :title="'信息编辑'"></el-step>
			<el-step title="图片上传"></el-step>
			<el-step title="图片下载"></el-step>
		</el-steps>

		<div class="infoEdit">
			<el-form v-if="active === 1">
				<!-- <el-form-item label="时间:" :label-width="formLabelWidth">
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
				</el-form-item> -->
				<el-form-item label="日期:" :label-width="formLabelWidth">
					<el-date-picker
						class="infoEditItem"
						v-model="dateValue"
						type="date"
						value-format="yyyy-MM-dd"
						placeholder="选择日期"
						@change="handleDateChange"
						:clearable="false"
					>
					</el-date-picker>
				</el-form-item>
				<el-form-item label="周几:" :label-width="formLabelWidth">
					<el-select
						class="infoEditItem"
						v-model="weekValue"
						disabled
						placeholder="请选择"
					>
						<el-option
							v-for="item in weekOptions"
							:key="item.value"
							:label="item.label"
							:value="item.value"
						>
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="地点:" :label-width="formLabelWidth">
					<el-input class="infoEditItem" v-model="add" placeholder="请输入" />
				</el-form-item>
			</el-form>
		</div>

		<!-- 上传组件 -->
		<div
			class="uploadBox transition-box"
			v-if="active === 2"
			v-show="downloading === false"
		>
			<el-upload
				class="upload-demo"
				action="https://jsonplaceholder.typicode.com/posts/"
				:on-remove="handleRemove"
				:file-list="fileList"
				list-type="picture"
				multiple
				:auto-upload="false"
				:on-change="handleChange"
				:limit="4"
				accept=".jpg,.png,.jpeg"
			>
				<el-button class="uploadBtn" type="primary" v-if="fileList < 1"
					>点击上传</el-button
				>
				<!-- <div slot="tip" class="el-upload__tip">只能上传jpg/png文件</div> -->
			</el-upload>
		</div>

		<!-- 下一步 按钮 -->
		<div class="nextStepBox">
			<el-button
				class="nextStepBtn"
				type="primary"
				@click="nextStep"
				v-show="
					active === 1 ||
					(active === 2 && fileList.length > 0 && downloading === false)
				"
				>{{ active === 1 ? '下一步' : '添加水印并下载' }}</el-button
			>
		</div>

		<!-- 图片预览区域 -->
		<div
			v-if="imageUrl !== ''"
			ref="vueDomSaveToImage"
			:class="imgDirection === '1' ? 'imgBox1' : 'imgBox0'"
		>
			<div :class="imgDirection === '1' ? 'imgCut0' : 'imgCut1'">
				<img class="saveImg" :src="imageUrl" alt="" />
			</div>

			<div class="timeValue">{{ timeValue }}</div>
			<div class="details">
				<span class="dateValue">{{ dateValue }}</span>
				<div class="empty"></div>
				{{ '星期' + weekValue + '' }}
				<div class="empty"></div>
				<img src="@/assets/ding.png" alt="" class="dingIcon" />
				{{ add }}
			</div>

			<img src="../assets/logo.png" class="logo" alt="" />
		</div>

		<!-- 提示 弹窗 -->
		<el-dialog title="温馨提示" :visible.sync="dialogVisible" width="40%">
			<div>
				每次最多可选择<span style="color: red">4</span
				>张图片，根据图片文件名自动匹配水印时间
			</div>
			<div>
				图片文件名为<span style="color: red">[会议照片]</span>时,时间随机取<span
					style="color: blue"
					>07:56~08:01</span
				>。
			</div>
			<div>
				图片文件名为<span style="color: red">[点名册、会议纪要、公告栏]</span
				>时,时间随机取<span style="color: blue">08:05~08:09</span>。
			</div>

			<span slot="footer" class="dialog-footer">
				<el-button type="primary" plain @click="dialogVisible = false"
					>确 定</el-button
				>
			</span>
		</el-dialog>
	</div>
</template>

<script>
import html2canvas from 'html2canvas';
import pkg from '../../package.json';

// 获取 min-max 之间的随机整数
function getRandomInt(min, max) {
	min = Math.ceil(min);
	max = Math.floor(max);
	return Math.floor(Math.random() * (max - min + 1)) + min;
}

export default {
	name: 'photoEdit',
	components: {},
	data() {
		return {
			formLabelWidth: '50px', //form 标签宽度
			dialogVisible: false, //弹窗
			count: 0,
			fileList: [], //上传文件列表
			downloading: false, //是否正在下载
			version: '', //版本号
			active: 1, //当前进度
			imageUrl: '', //预览图片
			fileName: '', //当前文件名
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
	mounted() {
		// 当前版本号
		console.log(
			'Version:' + `%c ${pkg.version} %c`,
			'background:#666;color:#fff;border-radius:3px;',
			''
		);
		this.version = pkg.version;
		// 获取当前日期 yyyy-mm-dd
		function timestampToTime(timestamp) {
			var date = new Date(timestamp); //时间戳为10位需*1000，时间戳为13位的话不需乘1000
			var Y = date.getFullYear() + '-';
			var M =
				(date.getMonth() + 1 < 10
					? '0' + (date.getMonth() + 1)
					: date.getMonth() + 1) + '-';
			var D =
				(date.getDate() < 10 ? '0' + date.getDate() : date.getDate()) + ' ';

			return Y + M + D;
		}
		this.dateValue = timestampToTime(Date.now());
		this.handleDateChange();
		this.dialogVisible = true; //打开提示弹窗
	},

	methods: {
		// 下一步
		nextStep() {
			if (this.active === 3) {
				this.active = 1;
			} else if (this.active === 2) {
				this.downloading = true;
				this.handleFileChange();
			} else {
				this.active++;
			}
		},

		// 删除图片
		handleRemove(file, fileList) {
			console.log(file, fileList);
			this.fileList = fileList;
		},

		// 上传图片修改
		handleChange(file, fileList) {
			this.fileList = fileList;
			console.log(file, fileList, 'change');
		},

		// 日期获取周几
		handleDateChange() {
			let datelist = ['日', '一', '二', '三', '四', '五', '六'];
			let week = datelist[new Date(this.dateValue).getDay()];
			this.weekValue = week;
		},

		// 上传
		handleFileChange() {
			var timer = setInterval(() => {
				this.export(this.fileList[this.count]);
				this.count += 1;
				if (this.count >= this.fileList.length) {
					clearInterval(timer);
				}
			}, 1000);
		},

		// 图片处理
		export(item) {
			const loading = this.$loading({
				lock: true,
				text: '下载中，请稍后...',
				spinner: 'el-icon-loading',
				background: 'rgba(0, 0, 0, 0.7)',
			});

			this.imageUrl = item.url;
			this.fileName = item.name.substring(0, item.name.lastIndexOf('.'));

			// 根据文件名 匹配时间
			let time = [
				'07:56',
				'07:57',
				'07:58',
				'07:59',
				'08:00',
				'08:01',
				'08:05',
				'08:06',
				'08:07',
				'08:08',
				'08:09',
				'08:07',
				'08:06',
				'08:05',
			];

			if (
				this.fileName == '会议内容' ||
				this.fileName == '会议纪要' ||
				this.fileName == '会议记录'
			) {
				this.timeValue = time[getRandomInt(0, 5)];
				this.imgDirection = '1';
			} else if (
				this.fileName == '点名册' ||
				this.fileName == '公告栏' ||
				this.fileName == '会议照片'
			) {
				this.timeValue = time[getRandomInt(6, 13)];
				this.imgDirection = '0';
			}

			this.$nextTick(() => {
				html2canvas(this.$refs.vueDomSaveToImage).then((res) => {
					console.log(res, 'res');
					let imgUrl = res.toDataURL('image/jpg');
					const save_link = document.createElementNS(
						'http://www.w3.org/1999/xhtml',
						'a'
					);
					save_link.href = imgUrl;
					save_link.download = this.fileName + '.jpg'; //保存文件名格式为 ***-点名册、会议纪要、等文件名
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
					this.$notify({
						title: this.fileName,
						message: '[' + this.fileName + ']' + '保存成功!',
						position: 'bottom-right',
						type: 'success',
						duration: 0,
					});
				});
			});
			if (this.count + 1 >= this.fileList.length) {
				this.$message({
					message: '全部处理完成，2秒后返回主页',
					type: 'success',
				});
				setTimeout(() => {
					loading.close();
					history.go(0);
				}, 2000);
			}
		},
	},
};
</script>

<style>
.edit {
	width: 90%;
	height: 100%;
	padding: 0 5%;
	margin: 0 auto;
	/* text-align: center; */
	/* display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center; */
}

.title {
	display: flex;
	align-items: center;
}
.tipsIcon {
	font-size: 30px;
	cursor: pointer;
	color: royalblue;
	margin-left: 10px;
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
.saveImg {
	width: 100%;
	height: 100%;
	object-fit: cover;
	/* position: absolute; */
}
.dateValue {
	font-size: 32px;
}
.dingIcon {
	width: 40px;
}
.empty {
	width: 10px;
}
.timeValue {
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

.infoEdit {
	width: 100%;
	height: 100%;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	margin-top: 50px;
}

.infoEditItem {
	width: 200px !important;
}

.upload-demo {
	width: 400px;
	margin: 0 auto;
	text-align: center;
}

.uploadBox,
.nextStepBox {
	width: 100%;
	height: 100%;
	margin: 0 auto;
	text-align: center;
	margin-top: 50px;
}

.nextStepBtn,
.uploadBtn {
	width: 200px;
	height: 50px;
}

.el-dialog {
	border-radius: 20px !important;
	margin-top: 30vh !important;
}
.el-dialog__title {
	font-weight: 600;
	font-size: 24px !important;
}
.el-dialog__body {
	font-size: 18px !important;
	line-height: 3em;
}
</style>
