<template>
	<div class="purse_user_type">
		<div class="typo">
			<blockquote class="blockquote_normal">
				<p>
					用户ID：<input class="mdui-textfield-input input_normal" type="text" v-model="keyword.user_id" />
					银行户名：<input class="mdui-textfield-input input_normal" type="text" v-model="keyword.realname" />
				</p>
				<p>
					日期筛选：<vue-datepicker-local v-model="keyword.date" clearable />
				</p>
				<a class="mdui-btn mdui-ripple mdui-color-theme" @click="search(1)"><i class="mdui-icon mdui-icon-left material-icons">search</i>搜索</a>
				<a class="mdui-btn mdui-ripple mdui-color-pink" @click="exports"><i class="mdui-icon mdui-icon-left material-icons">file_upload</i>导出</a>
			</blockquote>
			<blockquote class="blockquote_normal">
				<a class="mdui-btn mdui-ripple mdui-color-theme" @click="success_all"><i class="mdui-icon mdui-icon-left material-icons">check</i>批量同意</a>
			</blockquote>
		</div>
		<div class="mdui-table-fluid table-data-fluid">
			<table class="mdui-table mdui-table-hoverable table-data">
				<thead>
				<tr>
					<th><label class="mdui-checkbox"><input type="checkbox" /><i class="mdui-checkbox-icon" @click="check_all"></i></label></th>
					<th>#</th>
					<th>ID</th>
					<th>用户ID</th>
					<th>到账类型</th>
					<th>申请金额(分)</th>
					<th>手续费(分)</th>
					<th>打款金额(分)</th>
					<th>银行户名</th>
					<th>银行名</th>
					<th>银行卡号</th>
					<th>冻结ID</th>
					<th>成功转账ID</th>
					<th>申请状态</th>
					<th>备注</th>
					<th>创建时间</th>
					<th>修改时间</th>
					<th>操作</th>
				</tr>
				</thead>
				<tbody>
				
				<tr v-for="(val,key,index) in list.data">
					<td><label class="mdui-checkbox"><input type="checkbox" :value="val.id" v-model="success_all_id" /><i class="mdui-checkbox-icon"></i></label></td>
					<td v-text="key+1"></td>
					<td v-text="val.id"></td>
					<td v-text="val.user_id"></td>
					<td v-text="val.pay_type"></td>
					<td v-text="val.amount"></td>
					<td v-text="val.fee"></td>
					<td v-text="val.amount_actual"></td>
					<td v-text="val.realname"></td>
					<td v-text="val.bank_name"></td>
					<td v-text="val.bank_no"></td>
					<td v-text="val.freeze_id"></td>
					<td v-text="val.transfer_id"></td>
					<td v-html="status[val.status]"></td>
					<td v-text="val.remarks"></td>
					<td v-text="val.created_at"></td>
					<td v-text="val.updated_at"></td>
					<td>
						<div class="mdui-btn-group" v-if="val.status == 0">
							<a class="mdui-btn mdui-ripple mdui-color-theme" @click="success(val.id)">打款成功</a>
							<a class="mdui-btn mdui-ripple mdui-color-deep-orange" @click="fail(val.id)">打款失败</a>
						</div>
					</td>
				</tr>
				</tbody>
			</table>
		</div>
		<div class="mdui-color-white footer">
			<div class="mdui-color-grey-300 scrollbar" mdui-tooltip="{content:'便捷滚动条',position:'top'}"><div class="scrollbar-bar"></div></div>
			<pagination
					:pageInfo="{
						total:list.total,
						current:list.current_page,
						pagenum:list.per_page,
						page:list.last_page,
						pagegroup:9,
						skin:'#2196F3'
					}"
					@change="search"
			></pagination>
		</div>
	</div>
</template>
<script>
	export default {
		data(){
			return {
				list : [],
				purse_type : '',
				form : '',
				dialog : '',
				success_all_id : [],
				keyword : {
					page : 1,
					export : 0,
					user_id : '',
					realname : '',
					date : [],
				},
				status : [
					'申请中',
					'<span style="color:#16A05D">提现成功</span>',
					'<span style="color:#DE5145">提现失败</span>',
				]
			};
		},
		methods : {
			success(id){
				let t = this;
				mdui.confirm('确认后将从用户对应钱包扣除相应金额，确认请点击【确定】按钮', '已打款？', function(){
					post('/withdraw/success',{id:[id],type:''},function(){
						t.init();
					});
				},function(){},{history:false,confirmText:'确定',cancelText:'取消'});
			},
			fail(id){
				let t = this;
				mdui.prompt('标记为失败后，对应申请金额会原路返还给用户，知悉后请填写【失败原因】', '填写失败原因', function(value){
					if(value){
						post('/withdraw/fail',{id:id,remarks:value,type:''},function(){
							t.init();
						});
					}
					
				},function(){},{history:false,confirmText:'确定',cancelText:'取消'});
			},
			check_all(){
				let t = this;
				if(t.success_all_id.length >= t.list.data.length){
					t.success_all_id = [];
				}else{
					let success_all_id = [];
					$.each(t.list.data,function($k,$v){
						success_all_id.push($v.id);
					});
					t.success_all_id = $.unique(success_all_id);
				}
			},
			success_all(){
				let t = this;
				mdui.confirm('确认后将从用户对应钱包扣除相应金额，确认请点击【确定】按钮', '将进行批量打款成功操作', function(){
					let waiting = mdui.alert('请耐心等待批量作业完成，切勿关闭网页等操作。','批量处理中...',function(){},{history:false,confirmText:'',modal:true,closeOnEsc:false});
					post('/withdraw/success',{id:t.success_all_id,type:''},function(){
						t.init();
						waiting.close();
					},function(){
						waiting.close();
					});
				},function(){},{history:false,confirmText:'确定',cancelText:'取消'});
			},
			search(page){
				this.keyword.page = page;
				this.keyword.export = 0;
				this.init();
			},
			exports(){
				this.keyword.export = 1;
				this.init();
			},
			init(){
				let t = this;
				get('/withdraw/bank',t.keyword,function(data){
					t.list = data.data;
					t.purse_type = data.purse_type;
					if(t.keyword.export){
						mdui.alert('可在左侧【导出任务】菜单查看任务状态并下载文件','已放入导出任务',function(){},{history:false});
					}
				});
			}
		},
		mounted(){
			let t = this;
			t.init();
			let tableFluidLeft = $('.table-data-fluid')[0];
			let table_width = $('.table-data').width(),$scrollBar = $('.scrollbar');
			$('.scrollbar-bar').width(table_width);
			$scrollBar[0].addEventListener('scroll', function(){
				tableFluidLeft.scrollLeft = $scrollBar[0].scrollLeft;
			});
		}
	}
</script>
