<template>
  <div class="form-step">
      <FormKit
        v-model="choixTraiteur"
        name="choixTraiteurFait"
        type="radio"
        label="Avez-vous effectué le choix du traiteur avec un de nos conseillers ?"
        :options="['Oui', 'Non']"
        validation="required"
      />
      <FormKit
        v-if="choixTraiteur === 'Non'"
        v-model="traiteur"
        name="offert"
        type="radio"
        label="Désirez-vous offrir un goûter ?"
        :options="['Oui', 'Non']"
        validation="required"
      />
      <div v-if="choixTraiteur === 'Non' && traiteur === 'Oui'">
        <div v-if="formData.traiteur.value.intro">{{formData.traiteur.value.intro}}</div>
        <div class="flex sm:items-end items-start flex-col-reverse sm:flex-row sm:gap-3 pt-3">
          <div class="flex-1 w-full">
            <FormKit
              v-model="menu"
              name="menu"
              type="select"
              placeholder="Choisir un menu"
              label="Choix du menu"
              :options="menus"
              validation="required"
            />
          </div>
          <div v-if="formData.traiteur.value.menu">
            <a target="_blank" :href="formData.traiteur.value.menu">
              <FormKit type="button" prefix-icon="eye">Voir le menu</FormKit>
            </a>
          </div>
        </div>
        
        
        <template v-if="choix_additionnels">
          <div v-for="choix_additionnel in choix_additionnels" :key="choix_additionnel.identifiant">
            <FormKit type="group" name="selection">
              {{choix_additionnel.nom}} ({{choix_additionnel.nombre}})
              <FormKit :name="choix_additionnel.identifiant" type="list">
                <template v-for="i in parseInt(choix_additionnel.nombre)" :key="i">
                  <FormKit
                    name="choix"
                    type="select"
                    :label="choix_additionnel.nom"
                    :classes="{
                      label: '!hidden'
                    }"
                    placeholder="Faites un choix"
                    :options="choix_additionnel.choix.map(c => { return {value:c.nom, label: c.nom + (c.cout_additionnel ? ' - Coût additionnel de ' + moneyFormat(c.cout_additionnel) + '/personne' : ''), ...c} })"
                    validation="required"
                  />
                </template>
              </FormKit>
            </FormKit>
          </div>
        </template>
        <FormKit
          v-model="allergies"
          name="allergies"
          type="radio"
          label="Allergies à spécifier"
          :options="['Oui', 'Non']"
          validation="required"
        />
        <template v-if="allergies === 'Oui'">
          <FormKit
            type="textarea"
            name="allergiesSpecifiees"
            label="Spécifier les allergies"
            rows="4"
            validation="required"
          />
        </template>
        <FormKit
          v-model="alcool"
          name="alcool"
          type="radio"
          label="Avec ou sans alcool"
          :options="{
            Oui: 'Avec alcool',
            Non: 'Sans Alcool'
          }"
          validation="required"
        />
        <div v-if="alcool === 'Oui'" class="message_alcool" v-html="formData.traiteur.value.alcohol_permit_text"/></div>
  </div>
</template>

<script>
export default {
  name: 'Traiteur',
  data() {
    return {
      alcool: null,
      moneyFormatter: null,
      allergies: null,
      choixTraiteur: null,
      traiteur: null,
      menu: null
    };
  },
  methods:{
    log() {
      console.log(this);
    },
    moneyFormat(amount){
      return this.moneyFormatter.format(parseFloat(amount));
    }
  },
  computed: {
    menus() {
      return this.formData.traiteur.value.choix_de_menu.map(menu => {
        return {
          value: menu.nom,
          label:  `${menu.nom} - ${this.moneyFormat(menu.prix)}/personne`,
          ...menu
        };
      });
    },
    selectedMenu() {
      const {menu} = this;
      return menu 
        ? this.menus.find(item => item.value === menu) 
        : null;
    },
    choix_additionnels() {
      return this.selectedMenu && this.selectedMenu.choix_additionnels
        ? this.selectedMenu.choix_additionnels.map(choix => {
            const choix_additionnel = this.formData.traiteur.value.choix_additionnels.find(({identifiant}) => identifiant === choix.identifiant);
            return {
              ...choix,
              ...choix_additionnel
            };
        })
        : null;
    }
  },
  created(){
    this.moneyFormatter = new Intl.NumberFormat('fr-CA', {style:'currency', currency: 'CAD'});
  },
  props: {
    formData: {
      type: Object,
      required: true,
    },
  }
};
</script>