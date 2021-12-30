
<template>
<div>
    <v-card
    class="mx-auto"
    max-width="500"
    style="margin-top:50px; border-color:#9E9D24;"
    elevation="10"
  >
    <v-list shaped>
      <v-list-item-group
     
        multiple
      >
        <template v-for="(item) in api">
          <v-divider
            v-if="!item"
            :key="item.id"
          ></v-divider>

          <v-list-item
            v-else
            :key="item.id"
            :value="item"
            @click="clickfunc(item)"
            @dblclick="item.completed =!item.completed, togglecompleted(item)"
            v-bind:class="item.completed ? 'active' :''"
          >
            <template >
              <v-list-item-content>
                <v-list-item-title v-text="item.title"></v-list-item-title>
              </v-list-item-content>

              <!-- <v-list-item-action> -->
                 <v-checkbox
                  v-model="item.completed"
                  color = "#9E9D24"
                  @click="togglecompleted(item)"
                ></v-checkbox>
                
              <!-- </v-list-item-action> -->
            </template>
           
          </v-list-item>
        </template>
        
      </v-list-item-group>
    </v-list>
  </v-card>
  <div>
    <div style="display: flex; justify-content: center;">
      <v-btn class="btn" @click="add = !add, update=false"
        color="lime darken-2"
        elevation="2"
        small> 
        add task
        </v-btn>
      <v-btn class="btn" @click="deleteTask()" :disabled='disabledel'
       color="lime darken-2"
       elevation="2"
       small> 
       delete task
       </v-btn>
      <v-btn class="btn" @click="update = !update, add=false" :disabled='disableup'
       color="lime darken-2"
       elevation="2"
       small> update task</v-btn>
    </div>
      <div max-width="400px"  style="display:flex; justify-content: center;">
      <div v-show="update">
        <v-text-field
          style="width:300px; "
          v-model="title"
          label="title"
          required
        ></v-text-field>         
         <v-btn
            class="mr-4 lime darken-2"
            type="submit"
            :disabled="title==''? true: false"
            @click="updateTask()"
            small
        >
            submit
         </v-btn>
      </div >
      </div>
      <div max-width="400px" style="display: flex; justify-content: center;">
      <div v-show="add" >
        <v-text-field
          color="lime darken-3"
          style="width:300px;"
          v-model="title"
          label="title"
          required
        ></v-text-field>         
         <v-checkbox
          v-model="complete"
          color="lime darken-3"
          value="1"
          label="done?"
          type="checkbox"
        ></v-checkbox>
         <v-btn
            class="mr-4 lime darken-2"
            type="submit"
            :disabled="title==''? true: false"
            @click="addTask()"
            small
            
        >
            submit
         </v-btn>
      </div>
      </div>
  </div>
  </div>
</template>

<script>
export default {
    name: 'Main',
    data(){
        return{
            api:[],
            title:'',
            complete:false,
            deletedItems:[],
            add:false,
            update :false,
            array:[],
            updateItem : null,
            disabledel:true,
            disableup:true,
            active:false
        }
    },
 
    async mounted(){
       await fetch('https://jsonplaceholder.typicode.com/todos')
        .then(res=> res.json())
        .then(data=> this.api = data.slice(0,5))
    },

    methods:{
        async addTask(){
            let content = {
                userId:1, 
                title:this.title,
                completed:this.complete,
            }
            await fetch('https://jsonplaceholder.typicode.com/todos',
             {
                method:'post',
                headers:{
                    'Content-type':'application/json; charset=UTF-8'
                },
                body:JSON.stringify(content)
             }).then((res)=>res.json())
              //  .then(data=> [...this.api , data])
               .then((data)=> this.api.push(data))

        this.title = ''
        this.disabledel=true
        this.disableup=true
        this.update = false
            
        },
        async deleteTask(){
            let res 
            for(var item of this.deletedItems){
               res = (this.api.find(obj=>
                obj.id == item.id
              ))
            
              console.log(res);
             await fetch(`https://jsonplaceholder.typicode.com/todos/${res.id}`,
              {
                  method:'DELETE',
                  headers:{
                      'Content-type':'application/json'
                  },   
              });
           } 
          this.api = this.api.filter(item=> !this.deletedItems.includes(item))  
          this.deletedItems =[]
          this.disabledel=true
          this.disableup=true
          this.update = false

        },
        async updateTask(){
            this.updateItem.title = this.title
            const content={
              userId:this.updateItem.userId,
              completed:this.updateItem.completed,
              title:this.updateItem.title
            }
           await fetch(`https://jsonplaceholder.typicode.com/todos/${this.updateItem.id}`,
            {
                method:'PUT',
                headers:{
                    'Content-type':'application/json'
                },
                body:JSON.stringify(content)
            })

          let ind = this.api.findIndex(i => i.id == this.updateItem.id)
          this.api[ind].title = this.updateItem.title
          this.title = ''
          this.disabledel=true
          this.disableup=true 
          this.update = false

        },

        async togglecompleted(item){
          const content={
              userId: item.userId,
              completed: item.completed,
              title: item.title,
              id:item.id
            }
         await fetch(`https://jsonplaceholder.typicode.com/todos/${item.id}`,
            {
                method:'PUT',
                headers:{
                    'Content-type':'application/json'
                },
                body:JSON.stringify(content)
            }).then(res=>res.json())
            .then(data=> console.log(data))
            // this.api = this.api.map(task=> {
            //   task.id == item.id ? {
            //     ...task, completed : item.completed
            //   }: task
            // })
        },

       clickfunc(item){
         if(this.deletedItems.includes(item)){
           this.updateItem = null
           
           this.deletedItems.splice(this.deletedItems.indexOf(item),1)
           if(this.deletedItems.length == 0){
             this.disabledel = true
             this.disableup = true
           }
           if(this.deletedItems.length == 1){
              this.updateItem = this.deletedItems[0]
              this.disableup = false
           }
         }
         else{
           this.deletedItems.push(item)
           this.disabledel=false
           this.disableup=false
            this.updateItem = item
           if(this.deletedItems.length > 1){
              this.disableup=true
              this.updateItem = null
           }
         }
    },
}}
</script>
<style scoped>
.active{
  border-left-style: solid;
  border-left-color: #9E9D24;
  /* background-color: palevioletred; */
}
.btn{
  margin-right: 50px;
  margin-top: 10px;
}
</style>