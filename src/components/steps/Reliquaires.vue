<script setup>
  import { FormKitIcon } from '@formkit/vue'
  import FsLightbox from "fslightbox-vue/v3"
  import CryptoJS from 'crypto-js';
</script>

<template>
  <div class="form-step hide-step-actions hide-step-badge">
    <FsLightbox
      :toggler="lightboxVisible"
      type="image"
      :sources="[
        imagePreview
      ]"
    />
    <div v-html="formData.reliquaires.value['opt_reliquaire_intro']" />
    <div v-if="selected && Object.keys(selected).length > 0" class="cart rounded border border-stone-200 border-solid p-3 ml-3 fixed bg-white bottom-3 right-3 z-50 w-fit drop-shadow-lg">
      <div class="text-lg border-b border-stone-200 border-solid font-bold">Sélection reliquaires</div>
      <div class="max-h-36 overflow-auto flex flex-col gap-3 pt-1.5">
        <div class="cart-item min-h-fit	flex gap-3 items-center" v-for="(item, key, index) in selected" :key="index">
          <FormKit 
            name="reliquaires[]"
            type="hidden"
            :value="`${item.image_text} x${item.qty}`"
          />
          <FormKitIcon 
            class="!w-6 h-6 p-1 cursor-pointer bg-red-500 text-white rounded-full !self-center hover:bg-red-700" 
            icon="close" 
            @click="() => deleteItem(item)" 
          />
          <img :src="item.image_url.sizes.thumbnail" width="40" />
          <span class="text-xs flex-1">{{item.categorie.opt_reliquaire_categorie}} - {{item.image_text}}</span>
          <span class="qty">
            <FormKit
              v-model="selected[key].qty"
              type="number"
              step="1"
              placeholder="Qté"
              @change="({target: {value}}) => !value || value == 0 ?  deleteItem(item) : null"
            />
          </span>
        </div>
      </div>
    </div>
    <FormKit type="multi-step" tab-style="tab">
      <FormKit  type="step" :label="categorie.opt_reliquaire_categorie" v-for="(categorie, i) in formData.reliquaires.value['opt_reliquaire_categories']" :key="i">
        <div class="reliquaires-options grid lg:grid-cols-5 md:grid-cols-4 sm:grid-cols-3 min-[480px]:grid-cols-2 gap-3">
          <div class="bg-stone-500 flex flex-col relative reliquaire drop-shadow-md" v-for="(reliquaire, index) in categorie.opt_reliquaire_images" :key="index">
            <FormKitIcon 
              class="bg-gold cursor-pointer drop-shadow !md:w-6 !md:h-6 !h-8 !w-8 justify-center text-stone-100 hover:text-stone-300 block absolute top-3 right-0" 
              @click="() => viewImage(reliquaire.image_url.url)" 
              icon="eye" 
            />
            <img class="w-full text-white" :src="reliquaire.image_url.sizes.thumbnail" />
            <div class="flex grow items-center justify-between text-white">
              <span class="text-xs p-2">{{reliquaire.image_text}}</span>
              <FormKitIcon 
                class="!w-8 h-8 cursor-pointer text-stone-100 hover:text-stone-300" 
                icon="add" 
                @click="() => addItem({categorie, ...reliquaire})" 
              />
            </div>
          </div>
        </div>
      </FormKit>
    </FormKit>
  </div>
</template>

<script>

export default {
  name: 'Reliquaires',
  data() {
    return {
      imagePreview: null,
      lightboxVisible: false,
      selected: {}
    };
  },
  methods: {
    viewImage(url){
      this.imagePreview = url;
      this.lightboxVisible = true;
    },
    hash(item){
      const jsonString = JSON.stringify(item);
      return CryptoJS.SHA256(jsonString).toString();
    },
    addItem(item){
      const hash = this.hash(item);
      if(this.selected[hash]){
        this.selected[hash].qty++;
      }else{
        this.selected[hash] = {
          qty: 1,
          ...item
        }
      }
    },
    deleteItem({qty, ...item}){
      const hash = this.hash(item);
      delete this.selected[hash];
    },
  },
  props: {
    formData: {
      type: Object,
      required: true,
    },
  },
};
</script>