<template>
	<div class="contacts-root">
		<div class="contacts-head">
			<el-input
				v-model="searchContext"
				placeholder="Search"
				prefix-icon="el-icon-search"
				clearable
			/>
			<span class="el-icon-refresh-right contacts-refresh" @click="refresh"/>
		</div>

		<div class="contacts-content">
			<el-tabs v-model="activeName" :stretch="true">
				<el-tab-pane label="Friends" name="friends">
					<el-collapse v-model="activeFriendGroup">
						<el-collapse-item v-for="(v, i) in friendsAll"
						                  :title="`${v.name} (${v.friends.filter(e=>e.sc.includes(searchContext)).length})`"
						                  :name="i" :key="i">
							<ContactEntry
								v-for="i in v.friends"
								:key="i.uin"
								:id="i.uin"
								:remark="i.remark"
								:name="i.nick"
								v-show="i.sc.includes(searchContext)"
								@dblclick="$emit('dblclick', i.uin, i.remark)"
							/>
						</el-collapse-item>
						<ContactEntry
							v-for="i in friendsFallback"
							:key="i.uin"
							:id="i.uin"
							:remark="i.remark"
							:name="i.nick"
							v-show="i.sc.includes(searchContext)"
							@dblclick="$emit('dblclick', i.uin, i.remark)"
						/>
					</el-collapse>
				</el-tab-pane>
				<el-tab-pane label="Groups" name="groups">
					<ContactEntry
						v-for="i in groupsAll"
						:key="i.group_id"
						:id="-i.group_id"
						:remark="i.group_name"
						v-show="i.sc.includes(searchContext)"
						@dblclick="$emit('dblclick', -i.group_id, i.group_name)"
					/>
				</el-tab-pane>
			</el-tabs>
		</div>
	</div>
</template>

<script>
import {ipcRenderer} from 'electron'
import ContactEntry from './ContactEntry.vue'

export default {
	components: {ContactEntry},
	data() {
		return {
			activeName: 'friends',
			groupsAll: [],
			/**
			 * @type GroupOfFriend[]
			 */
			friendsAll: [],
			searchContextEdit: '',
			activeFriendGroup: [],
			friendsFallback: [],
		}
	},
	computed: {
		searchContext: {
			get() {
				return this.searchContextEdit
			},
			set(val) {
				this.searchContextEdit = val.toUpperCase()
			},
		},
	},
	created() {
		setTimeout(() => ipcRenderer.invoke('getFriendsAndGroups')
			.then(({friends, groups, friendsFallback}) => {
				this.friendsAll = friends ? Object.freeze(friends) : null
				this.groupsAll = Object.freeze(groups)
				friendsFallback && (this.friendsFallback = Object.freeze(friendsFallback))
			}), 10 * 1000)
	},
	methods: {
		refresh() {
			this.friendsAll = this.groupsAll = this.friendsFallback = []
			ipcRenderer.invoke('getFriendsAndGroups')
				.then(({friends, groups, friendsFallback}) => {
					this.friendsAll = friends ? Object.freeze(friends) : null
					this.groupsAll = Object.freeze(groups)
					friendsFallback && (this.friendsFallback = Object.freeze(friendsFallback))
					this.$message.success('已刷新')
				})
		},
	},
}
</script>

<style>
.el-collapse-item__header {
	padding-left: 12px;
}

.el-collapse-item__content {
	padding-bottom: 0;
}

.el-collapse-item__wrap > div > div:last-child > div > div {
	border-bottom: unset !important;
}

.el-tabs__header {
	margin: unset !important;
}

.el-tabs__item {
	padding: 0;
}

.contacts-root {
	border-right: 1px solid #e1e4e8;
	height: 100vh;
	display: flex;
	flex-direction: column;
}

.contacts-head {
	margin: 0 12px;
	height: 64px;
	min-height: 64px;
	display: flex;
	align-items: center;
}

.contacts-content {
	overflow: auto;
}

.contacts-refresh {
	cursor: pointer;
	font-size: larger;
	color: #909399;
	margin-left: 10px;
}
</style>
