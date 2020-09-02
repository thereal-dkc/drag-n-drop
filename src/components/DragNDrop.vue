<template>
    <section id="MainContainer">
        <section class="list" id="StandbyList">
            <div v-for="item in standbyItemList" :key="item.id"
                class="listItemRenderer"
                draggable="true"
                @dragstart="dragStart($event, item)"
                @dragend="dragEnd($event, item, false)">
                {{item.name}}
            </div>
        </section>
        <section class="list" id="UseList"
             @drop="drop( $event, 1)"
             @dragover="dragOver"
             @dragenter.prevent>
            <div v-for="item in useItemList" :key="item.id"
                 class="listItemRenderer"
                 :data-id="item.id"
                 draggable="true"
                 @dragstart="dragStart($event, item)"
                 @dragend="dragEnd($event, item, true)">
                {{item.name}} ({{item.index}})
            </div>
        </section>
    </section>
</template>

<script>
    export default {
        name: 'DragNDrop',
        data: () =>({
            standbyItemList: [],
            useItemList: [],
            dragBox: {
                height: 0,
                top: 0,
                bottom: 0,
                offsetX: 0,
                offsetY: 0,
            },
            dropZoneId: -1,
            dropUnderId: -1,
            droppedItemId: -1,
        }),
        methods: {
            getListItemRenderer() {
                return document.getElementsByClassName('listItemRenderer')[0];
            },
            dragStart(e, item) {
                const rect = this.getListItemRenderer().getBoundingClientRect();
                this.dragBox.height = rect.height;
                this.dragBox.offsetX = e.offsetX;
                this.dragBox.offsetY = e.offsetY;
                this.dropZoneId = -1;
                e.dataTransfer.effectAllowed = 'copy';
                e.dataTransfer.setData('itemId', item.id);
                this.droppedItemId = -1;
            },
            dragEnd(e, item, isDelete) {
                e.preventDefault();
                if (isDelete && this.dropZoneId !== 1 ) {
                    const itemId = item.id;
                    const isUseItem = this.useItemList.some(item => item.id === itemId);
                    if (isUseItem) {
                        this.useItemList.splice(item.index, 1);
                        this.indexingItem(this.useItemList);
                    }
                }
            },
            dragOver(e) {
                e.preventDefault();
                e.dataTransfer.dropEffect = 'copy';
                //detect drag item under item
                this.detectUnderItemId(e.clientX, e.clientY);
            },
            drop(e, listId) {
                e.preventDefault();
                this.dropZoneId = listId;
                this.dropItem(Number(e.dataTransfer.getData('itemId')));
            },
            dropItem(itemId) {
                this.droppedItemId = itemId;
                if (this.dropUnderId === itemId) return;

                const isUseItem = this.useItemList.some(item => item.id === itemId);
                const item = this.standbyItemList.find(item => item.id === itemId);

                if (isUseItem) {
                    this.useItemList.splice(item.index, 1);
                }
                if (this.dropUnderId > -1 ) {
                    const underItem = this.useItemList.find(item => item.id === this.dropUnderId);
                    this.useItemList.splice( underItem.index, 0, item);
                } else {
                    this.useItemList.push(item);
                }
                this.indexingItem(this.useItemList);
            },
            detectUnderItemId(x, y) {
                const useItems = document.getElementById('UseList').childNodes;
                if(useItems===undefined) return;
                this.dragBox.top = y - this.dragBox.offsetY;
                this.dragBox.bottom = this.dragBox.top + this.dragBox.height;
                this.dropUnderId = this.findDropUnderId(
                    this.findUnderItems(useItems, this.dragBox.top, this.dragBox.bottom)
                );
            },
            findUnderItems(listItems, top, bottom) {
                const underItem = [];
                listItems.forEach(item => {
                    const itemRect = item.getBoundingClientRect();
                    if ((itemRect.top >= top && itemRect.top <= bottom)
                        || (itemRect.bottom >= top && itemRect.bottom <= bottom)) {
                        underItem.push({
                            id: Number(item.dataset.id),
                            top: itemRect.top,
                            bottom: itemRect.bottom,
                        });
                    }
                });
                return underItem;
            },
            findDropUnderId(underItem) {
                if (underItem.length > 1) {
                    if (Math.abs(underItem[0].top - this.dragBox.top)
                        < Math.abs(underItem[1].top - this.dragBox.top)) {
                        return underItem[0].id;
                    } else {
                        return underItem[1].id;
                    }
                } else if (underItem.length === 1 && this.dragBox.top - underItem[0].top < this.dragBox.height/2) {
                    return underItem[0].id;
                } else {
                    return -1;
                }
            },
            indexingItem(arr) {
                for (let i=0; i < arr.length; i ++) {
                    arr[i].index = i;
                }
            },
        },
        watch:  {
        },
        created() {
            for (let i = 0; i < 10; i ++) {
                const item = { id: i , name: 'Item '+ i, index: i, style:'dragEnd' };
                this.standbyItemList[i] = item;
            }
        },
        mounted() {
            if (document.getElementById('DragBox')) {
                this.dragBox = document.getElementById('DragBox');
            }
        },
        updated() {
        },
    };
</script>

<style scoped>
    #MainContainer {
        margin: auto;
        width: 80%;
        height: 80%;
        border: 1px solid darkgreen;
        display: flex;
        align-items: center;
        justify-content: space-around;
    }
    .list {
        border: 1px solid #000000;
        background-color: #E3E3E3;
        width: 30%;
        height: 500px;
        overflow: auto;
    }
    .listItemRenderer {
        border: 1px solid #000000;
        background-color: #f6f6f6;
        width: 96%;
        height: 50px;
        margin: 2px auto;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .listItemRenderer:hover {
        border: 1px solid #ff7700;
        background-color: #ffffff;
        box-shadow: 3px 4px 2px rgba(127, 101, 101, 0.4);
        -webkit-transition-property: background-color, border;
        -webkit-animation-duration: 0.4s;
        -webkit-transition-timing-function: ease;
        transition-property: background-color, border;
        transition-duration: 0.2s;
        transition-timing-function: ease;
    }
    #DragBox {
        /*visibility: hidden;*/
        pointer-events: none;
        position: absolute;
        z-index: 9999;
    }
</style>