<template>
  <div class="box container flex flex-row">
    <div class="w-1/5 mr-4">
      <button @click="stop()" class="button danger w-full mb-2">
        Hentikan Permainan
      </button>
      <jumper
        :total="permainanState.soalCount"
        :value="1"
        v-model="soalID"
      ></jumper>
    </div>
    <soal
      v-if="soal != undefined"
      :soal="soal"
      teksSubmit="Kirim Jawaban"
      @submit="soalSubmit"
      @change="soalChange"
      ref="soalView"
      class="w-4/5 ml-4"
    >
      <p
        v-if="permainanState.interaktif"
        v-show="lastJawabanState > 0"
        class="mb-4 text-white font-semibold p-4"
        :class="lastJawabanState == 2 ? 'bg-green-500' : 'bg-red-500'"
      >
        Jawaban anda {{ lastJawabanState == 2 ? "benar" : "salah coba lagi." }}
      </p>
    </soal>
  </div>
</template>

<script>
import { Permainan } from "@/api.js";
import Soal from "../../soal/components/Soal.vue";
import Jumper from "../components/Jumper.vue";

export default {
  components: {
    Soal,
    Jumper
  },
  data() {
    return {
      permainanStarted: false,
      permainanState: {
        interaktif: false,
        soalCount: 0,
        jawabanCount: 0
      },
      soal: null,
      soalID: 0,
      lastJawabanState: 0
    };
  },
  methods: {
    soalSubmit(co) {
      let jawaban = co.pilihanTerpilih;
      let id = co.soal.id;
      Permainan.putJawaban(id, jawaban).then(val => {
        if (this.permainanState.interaktif) {
          this.lastJawabanState = val.benar ? 2 : 1;
          setTimeout(() => {
            this.lastJawabanState = 0;
            if (val.benar) {
              this.nextSoal();
            }
          }, 1000);
        } else {
          this.nextSoal();
        }
      });
    },
    nextSoal() {
      if (this.soalID < this.permainanState.soalCount) this.soalID += 1;
    },
    soalChange(soal) {
      this.$nextTick(function() {
        soal.pilihanTerpilih = -1;
      });
    },
    updateState() {
      Permainan.state().then(val => {
        this.permainanStarted = val.permainanStarted;

        if (this.permainanStarted) {
          this.permainanState = val.permainan;
          this.soalID = 1;
        }
      });
    },
    stop() {
      Permainan.stopPermainan().then(val => {
        this.$store.commit("setPermainanResult", val);
        this.$router.push("/permainan/result");
      });
    }
  },
  watch: {
    soalID() {
      if (this.permainanStarted) {
        Permainan.getSoal(this.soalID - 1).then(val => {
          this.soal = val;
        });
      }
    }
  },
  mounted() {
    this.updateState();
  }
};
</script>

<style>
</style>
