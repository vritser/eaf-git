<template>
  <div class="box">
    <Dialog title="Stash">
      <div
        v-if="stashInfo.length > 0"
        class="list"
        ref="stashs">
        <div
          v-for="info in stashInfo"
          :key="info.index"
          class="stash-item">
          <div
            class="stash-id"
            :style="{ 'color': idColor }">
            {{ info.id.slice(0, 7) }}
          </div>
          <div
            class="stash-index"
            :style="{ 'color': indexColor }">
            stash@[{{ info.index }}]
          </div>
          <div class="stash-message">
            {{ info.message }}
          </div>
        </div>
      </div>
      <div
        v-else
        class="notify">
        No stash found in current repository.
      </div>
    </Dialog>
  </div>
</template>

<script>
 import Dialog from "./Dialog.vue"

 export default {
   name: 'Stash',
   components: {
     Dialog
   },
   props: {
     stashInfo: Array,
     idColor: String,
     indexColor: String,
     dateColor: String,
     backgroundColor: String,
     selectColor: String,
     currentStashIndex: Number,
     pyobject: Object
   },
   data() {
     return {
       currentCommitId: ""
     }
   },
   watch: {
     currentStashIndex: {
       // eslint-disable-next-line no-unused-vars
       handler: function(val, oldVal) {
         if (this.stashInfo.length > 0) {
           this.currentCommitId = this.stashInfo[val].id;
           this.updateItemBackground(oldVal, val);
           this.keepSelectVisible();
         }
       }
     },
   },
   mounted() {
     var that = this;

     this.showHighlightLine();

     this.$root.$on("stash_view_diff", function () {
       that.pyobject.show_stash_diff(that.stashInfo[that.currentStashIndex].index);
     });

     this.$root.$on("stashSearchForward", function () {
       that.pyobject.stash_search_forward();
     });

     this.$root.$on("stashSearchBackward", function () {
       that.pyobject.stash_search_backward();
     });

     this.$root.$on("stash_apply", function () {
       that.pyobject.stash_apply(that.stashInfo[that.currentStashIndex].index, that.stashInfo[that.currentStashIndex].message);
     });

     this.$root.$on("stash_pop", function () {
       that.pyobject.stash_pop(that.stashInfo[that.currentStashIndex].index, that.stashInfo[that.currentStashIndex].message);
     });

     this.$root.$on("stash_drop", function () {
       that.pyobject.stash_drop(that.stashInfo[that.currentStashIndex].index, that.stashInfo[that.currentStashIndex].message);
     });
   },
   beforeDestroy() {
     this.$root.$off("js_stash_view_diff");
     this.$root.$off("stashSearchForward");
     this.$root.$off("stashSearchBackward");
     this.$root.$off("js_stash_apply");
     this.$root.$off("js_stash_pop");
     this.$root.$off("js_stash_drop");
   },
   methods: {
     showHighlightLine() {
       if (this.stashInfo.length > 0) {
         if (this.currentStashIndex !== null && this.currentStashIndex >= 0 && this.$refs.stashs.children.length > 0) {
           this.$refs.stashs.children[this.currentStashIndex].style.background = this.selectColor;
         }

         this.keepSelectVisible();
       }
     },

     updateItemBackground(oldIndex, newIndex) {
       if (oldIndex !== null && oldIndex >= 0) {
         var oldItem = this.$refs.stashs.children[oldIndex];
         oldItem.style.background = this.backgroundColor;
       }

       if (newIndex !== null && newIndex >= 0) {
         var newItem = this.$refs.stashs.children[newIndex];
         newItem.style.background = this.selectColor;
       }
     },

     keepSelectVisible() {
       /* Use nextTick wait DOM update, then make sure current file in visible area. */
       this.$nextTick(function() {
         var selectStash = this.$refs.stashs.children[this.currentStashIndex]
         if (selectStash !== undefined) {
           selectStash.scrollIntoView({behavior: "smooth", block: "end", inline: "end"});
         }
       })
     },
   }
 }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
 .box {
   width: 100%;
   height: 100%;
 }

 .stash-item {
   padding-left: 10px;
   padding-right: 10px;
   font-size: 16px;
   height: 30px;

   display: flex;
   flex-direction: row;
   align-items: center;
 }

 .stash-id {
   width: 80px;
 }

 .stash-message {
   padding-left: 20px;
   flex: 1;

   overflow: hidden;
   white-space: nowrap;
   text-overflow: ellipsis;
 }

 .highlight-line {
   width: 100%;
   z-index: 10;
   position: fixed;
 }

 .list {
   z-index: 100;
   max-height: calc(100vh - 100px);
   overflow-y: scroll;
 }
 
 .notify {
   display: flex;
   flex-direction: column;
   justify-content: center;
   
   height: 100%;
   
   font-size: 16px;
   text-align: center;
   justify-self: center;
   margin: auto;
   font-size: 20px;
 }
</style>
