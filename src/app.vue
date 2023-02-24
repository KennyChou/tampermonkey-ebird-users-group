<template>
  <div id="share-groups-fieldset">
    <div>
      <div :hidden="editMode">
        <span class="Heading Heading--h6" for="share-group">
          <span class="Heading-main">群組通訊錄</span>
        </span>
        <div id="share-group" class="u-stack-sm" style="background-color: white; max-height: 8rem; overflow-y:auto; box-shadow: inset 0 1px 2px rgba(33,33,33,.1); border: 1px solid #d3d3d3;">
          <template v-if="groups.length==0">
            <a class="u-inset-squish-sm" style="display:block; border-bottom: 1px solid #efefef;" @click="addAll" href="#share-recipients">全部成員</a>
          </template>
          <template v-else>
            <template v-for="(group, idx) in groups">
              <span class="u-inset-squish-sm" style="display: block; border-bottom: 1px solid #efefef;">
                <a @click="shareGroup(group.ebirdIds)" href="#share-recipients">{{ group.name }}</a>
                <a @click="editGroup(idx)" style="float: right;" href="#share-groups-fieldset">編輯</a>
              </span>
            </template>
          </template>
          
        </div>
        <p class="u-text-2"><a @click="editGroup(-1)" href="#share-groups-fieldset">新增群組</a></p>
      </div>
      <div id="user-group-editor" :hidden="!editMode">
        <span class="Heading Heading--h6" for="share-group">
          <span class="Heading-main">編輯群組</span>
        </span>
        <input class="u-text-2"  type="text" v-model="editData.name" />
        <span class="Heading-sub Heading-sub--inline">(用戶名稱或電子郵件地址，以逗號分隔)</span>
        <textarea class="u-text-2" v-model="editData.ebirdIds" id="ebird-users-group"></textarea>
        <div id="share-contacts" class="u-stack-sm" style="background-color: white; max-height: 8rem; overflow-y:auto; box-shadow: inset 0 1px 2px rgba(33,33,33,.1); border: 1px solid #d3d3d3;">
          <template v-for="user in users">
            <a class="u-inset-squish-sm" style="display:block; border-bottom: 1px solid #efefef;" @click="appendGroup(user)" href="#share-groups-fieldset">{{ user.name }}</a>
          </template>
        </div>
        <div class="ButtonGroup" style="margin-bottom: 20px;">
          <button class="Button Button--small Button--highlight u-inline-sm" @click="saveGroups" :disabled="this.editData.name.trim()=='' || this.editData.ebirdIds.trim()==''">存檔</button>
          <button class="Button Button--small Button--secondary u-inline-sm" @click="editMode=false">取消</button>
          <button class="Button Button--small Button--secondary" @click="deleteGroup" v-if="editId!=-1">刪除</button>
        </div>
        <span>&nbsp;</span>
      </div>
    </div>
  </div>
</template>

<script>
import { isDev } from './config'

export default {
  name: 'app',
  data() {
    return {
      editMode: false,
      users:[],
      editId:-1,
      groups:[],
      editData: {name: '', ebirdIds:''}
    }
  },
  mounted(){
    if(isDev){
      this.users.push({name: 'kenny', id: '123213123'})
      this.users.push({name: 'kenny123', id: 'kkk123213123'})
    }else{
      const useritems = document
          .getElementById('share-contacts')
          .getElementsByClassName(
            'u-inset-squish-sm'
          )
      for(const item of useritems){
        this.users.push({ name: item.text, id: item.getAttribute('onclick', 0).toString().split("'")[1]})
      }
    }
    try{
      this.groups = JSON.parse(localStorage.getItem('ebird-groups') || '[]')
    }catch(e){
      console.log(e)
    }
    // console.log(this.users)
  },
  methods: {
    addAll(){
      document.getElementById('share-recipients').value=this.users.map(user=>user.id).join(',')
    },
    editGroup(id){
      this.editId=id
      if(id==-1){
        this.editData.name='群組名稱'
        this.editData.ebirdIds=''
      }else{
        this.editData.name=this.groups[id].name
        this.editData.ebirdIds=this.groups[id].ebirdIds
      }
      this.editMode=true
    },
    appendGroup(user){
      let users
      if(this.editData.ebirdIds.trim()!=''){
        users=this.editData.ebirdIds.split(',').map(item=> item.trim())
      }else{
        users=[]
      }
        
      // console.log(users)
      if (!users.includes(user.id)){
        users.push(user.id)
      }
      this.editData.ebirdIds=users.join(',')

    },
    saveGroups(){
      if(this.editId==-1){
        this.groups.push({name: this.editData.name, ebirdIds: this.editData.ebirdIds})
      }else{
        this.groups[this.editId].name=this.editData.name
        this.groups[this.editId].ebirdIds=this.editData.ebirdIds
      }
      localStorage.setItem('ebird-groups', JSON.stringify(this.groups))
      this.editId=-1
      this.editMode=false
    },
    deleteGroup(){
      this.groups.splice(this.editId, 1)
      localStorage.setItem('ebird-groups', JSON.stringify(this.groups))
      this.editId=-1
      this.editMode=false
    },
    shareGroup(ebirdIds){
      document.getElementById('share-recipients').value=ebirdIds
    }
  },
}
</script>
