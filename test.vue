<template>
	<div class="test">
		<div class="desc">{{test.desc}}</div>
		<div class="questions">
			<div class="question" v-for="(index,item1) in test.questions">
				<h3>{{index+1}}.{{item1.title}} <span v-if="item1.isMultiselect">(多选)</span></h3>
				<p  v-for="(index,item2) in item1.options" 
						@click="selectAnswer(item1,item2)" 
						:class="[{'selected-blue':item2.selectState==1},{'selected-red':item2.selectState==2},{'selected-white':item2.selectState==0}]">{{item2.text}}</p>
				<p class="answer" v-if="test.answerState">正确答案：<span v-for="item3 in item1.answers">{{item3}}</span></p>
			</div>
			<div class="question-mark" v-if="test.answerState"></div>
		</div>
		<button @click="submitTest">{{test.submitBtn}}</button>
	</div>
</template>
<script type="text/babel">
import { subTest , getQuetions } from 'services/calendar-week/calendar-week';
export default {
	data () {
		return {
			test:{
				desc:'测试中所指的【开发者】包括：产品经理、UI设计、项目顾问、软件工程师、测试工程师。开发者是一个统称。',
				questions: getQuetions,
				answerState:false,
				isPassTest:true,
				submitBtn:'提交',
			}
		}
	},
	methods:{
		selectAnswer (item1,item2) { //选择答案
			if(item1.isMultiselect) { //多选
				if(item2.selectState == 0) {
					item2.selectState = 1;
					item1.userAnswewrs.push(item2.text.charAt(0));
				}else if(item2.selectState == 1){
					item2.selectState = 0;
					for(let i = 0, j = item1.userAnswewrs.length;i < j; i++) {
						if(item1.userAnswewrs[i] == item2.text.charAt(0)) {
							item1.userAnswewrs.splice(i,1)
						}
					}
				}
			}else { //单选
				item1.options.forEach(item22 => item22.selectState = 0);
				item2.selectState = 1;
				item1.userAnswewrs[0] = item2.text.charAt(0);
			}
		},
		submitTest () { //提交测试
			let self = this;
			self.test.answerState = !self.test.answerState;
			if(self.test.answerState){
				self.test.submitBtn = '重填答题';
				self.test.questions.forEach(item1 => {//漏选
					if(item1.userAnswewrs.length < 1) {
						self.test.isPassTest = false;
					}
				})
				self.test.questions.forEach(item1 => { // 答错飙红
					if(item1.answers.toString() != item1.userAnswewrs.toString()) {
						self.test.isPassTest = false;
						item1.userAnswewrs.forEach(item2 => {
							item1.options.forEach(item3 => {
								if(item3.text.charAt(0) == item2) {
									item3.selectState = 2;
								}
							})
						})
					}
				})
				self.test.isPassTest = self.test.questions.every(item1 => {// 全部答对
					return item1.answers.toString() == item1.userAnswewrs.toString()
				})

				if(self.test.isPassTest) { //全部答对
					self.$Message.success('认证成功！',3);
					self.test.answerState = false;
					subTest(self.$route.query.userId);
				}
			}else {
				this.initTest();
			}
		},
		initTest () {
			this.test.submitBtn = '提交';
			this.test.questions.forEach(item1 => {
				item1.options.forEach(item2 => {
					item2.selectState = 0;
				});
				item1.userAnswewrs = [];
			})
		}
	},
	destroyed () {
		this.initTest();
		this.test.answerState = false;
	}
}
</script>
<style lang="stylus" scoped>
	@import '~assets/css/var.styl'
	.test{
		padding-bottom 30px
	}
	.desc,h3,p {
		fpadding(8px)
		text-align justify
	}
	.desc,.question {
		fborder()
	}
	.questions {
		z-index 1
		position relative
	}
	.question-mark {
		z-index 2
		position absolute
		top 0
		left 0
		height 100% 
		width 100%
	}
	h3 {
		color $ColorBlueOperate
		font-size 18px
	}
	p {
		fborder(1px)
		transition all .5s ease
		&.selected-blue {
			background $ColorBlueOperate
			color #fff
			transition all .5s ease
		}
		&.selected-red {
			background #f00
			color #fff
			transition all .5s ease
		}
		&.selected-white {
			background #fff
			color #666
			transition all .5s ease
		}
	}
	.answer {
		color #f00
		span {
			padding 0 5px
		}
	}
	button {
		commonBtn()
	}
</style>