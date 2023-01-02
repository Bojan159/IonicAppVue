<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Ionic</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Ionic</ion-title>
        </ion-toolbar>
      </ion-header>
    </ion-content>
    <div style="max-width: 400px">
      <ion-list @submit.prevent="submitform">
        <ion-item>
          <ion-label position="floating">Marka</ion-label>
          <ion-input
            placeholder="Unesi marku"
            v-model="state.formdata.marka"
          ></ion-input>
        </ion-item>
        <ion-item>
          <ion-label position="floating">Model</ion-label>
          <ion-input
            placeholder="Unesi model"
            v-model="state.formdata.model"
          ></ion-input>
        </ion-item>
        <ion-item>
          <ion-label position="floating">Komponente</ion-label>
          <ion-input
            placeholder="Unesi komponente"
            v-model="state.formdata.komponente"
          ></ion-input>
        </ion-item>
        <ion-button button-type="submit" @click="submitform()">
          Submit
        </ion-button>
        <ion-button button-type="reset" @click="onReset()">Reset</ion-button>
      </ion-list>
    </div>
    <div style="max-width: 400px" v-if="state.listaLaptopa.length > 0">
      <ion-card v-for="laptop in state.listaLaptopa" :key="laptop.id">
        <ion-card-header>
          <ion-card-title>Marka: {{ laptop.marka }}</ion-card-title>
          <ion-card-title>Model: {{ laptop.model }}</ion-card-title>
          <ion-card-title>Komponente: {{ laptop.komponente }}</ion-card-title>
        </ion-card-header>
        <ion-button fill="clear" @click="deleteLaptope(laptop.id)"
          >Delete</ion-button
        >
        <ion-button fill="clear" id="open-modal" @click="setEdit(laptop)"
          >Edit</ion-button
        >
        <ion-modal
          :is-open="state.edit"
          id="edit"
          trigger="open-modal"
          :backdrop-dismiss="false"
        >
          <ion-list>
            <ion-item>
              <ion-label position="floating">Marka</ion-label>
              <ion-input
                placeholder="Unesi marku"
                v-model="state.editLaptopaPodaci.marka"
              ></ion-input>
            </ion-item>
            <ion-item>
              <ion-label position="floating">Model</ion-label>
              <ion-input
                placeholder="Unesi model"
                v-model="state.editLaptopaPodaci.model"
              ></ion-input>
            </ion-item>
            <ion-item>
              <ion-label position="floating">Komponente</ion-label>
              <ion-input
                placeholder="Unesi komponente"
                v-model="state.editLaptopaPodaci.komponente"
              ></ion-input>
            </ion-item>
            <ion-button
              type="submit"
              @click="
                editForm();
                state.edit = false;
              "
            >
              Spremi
            </ion-button>
            <ion-button @click="state.edit = false">Zatvori</ion-button>
          </ion-list>
        </ion-modal>
      </ion-card>
    </div>
  </ion-page>
</template>

<script>
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonInput,
  IonModal,
  IonButton,
  IonList,
  IonItem,
  IonCard,
  IonCardTitle,
  IonLabel,
  IonCardHeader,
} from "@ionic/vue";
import { defineComponent, reactive, onMounted, onUnmounted } from "vue";
import {
  addDoc,
  collection,
  query,
  onSnapshot,
  deleteDoc,
  doc,
  updateDoc,
} from "@firebase/firestore";
import { db } from "../main";

export default defineComponent({
  name: "HomePage",

  setup() {
    const state = reactive({
      formdata: {
        marka: "",
        model: "",
        komponente: "",
      },
      poruka: "",
      listaLaptopa: [],
      edit: false,
      editLaptopaPodaci: null,
    });

    async function submitform() {
      await addDoc(collection(db, "laptop"), {
        marka: state.formdata.marka,
        model: state.formdata.model,
        komponente: state.formdata.komponente,
      });
      state.formdata.marka = " ";
      state.formdata.model = " ";
      state.formdata.komponente = " ";
    }

    const editForm = async () => {
      await updateDoc(doc(db, "laptop", state.editLaptopaPodaci.id), {
        marka: state.editLaptopaPodaci.marka,
        model: state.editLaptopaPodaci.model,
        komponente: state.editLaptopaPodaci.komponente,
      });
    };

    let unsubscribe;

    const dohvatiLaptope = async () => {
      const q = query(collection(db, "laptop"));
      unsubscribe = onSnapshot(q, (querySnapshot) => {
        state.listaLaptopa = [];
        querySnapshot.forEach((doc) => {
          let laptop = {
            id: doc.id,
            marka: doc.data().marka,
            model: doc.data().model,
            komponente: doc.data().komponente,
          };

          state.listaLaptopa.push(laptop);
        });
      });
    };
    const deleteLaptope = async (id) => {
      await deleteDoc(doc(db, "laptop", id));
    };

    const setEdit = (laptop) => {
      state.edit = true;
      state.editLaptopaPodaci = {
        id: laptop.id,
        marka: laptop.marka,
        model: laptop.model,
        komponente: laptop.komponente,
      };
    };

    onMounted(() => {
      dohvatiLaptope();
    });

    onUnmounted(() => {
      unsubscribe();
    });

    const onReset = () => {
      state.formdata.marka = " ";
      state.formdata.model = " ";
      state.formdata.komponente = " ";
    };

    return {
      state,
      onReset,
      submitform,
      deleteLaptope,
      editForm,
      setEdit,
    };
  },
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonInput,
    IonModal,
    IonButton,
    IonList,
    IonItem,
    IonCard,
    IonCardTitle,
    IonLabel,
    IonCardHeader,
  },
});
</script>

<style scoped>
#container {
  text-align: center;

  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;

  color: #8c8c8c;

  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
