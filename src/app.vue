<template>
  <div class="app">
    <!-- add csv -->
    <label
      class="dropzone"
      :class="{ disabled: loading && loading.done !== loading.total }"
    >
      <span> + CSV </span>
      <input type="file" ref="input" hidden />
    </label>

    <!-- loading -->
    <div v-if="loading" class="loading">
      <span v-if="completed"> Completed successfully! </span>
      <span v-else-if="loading.total">
        <span> Loading </span>
        <span >
          {{ Math.round((loading.done / loading.total) * 100) + '%' }} -----
        </span>
        
        <span> {{ loading.done }} of {{ loading.total }} </span>
      </span>
    </div>

    <!-- download all -->
    <button
      v-if="products.length && loading.done === loading.total"
      v-on:click="downloadAll"
      class="btn"
    >
      download all
    </button>

    <!-- list -->
    <div class="list">
      <div v-for="product in products" :key="product.sku" class="card">
        <Display :product="product" :ref="setRef" />
      </div>
    </div>
  </div>
</template>
<script>
import Papa from 'papaparse';
import JSZip from 'jszip';
import { saveAs } from 'file-saver';
import Display from './components/display';

const sleep = ms => {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
};

export default {
  components: {
    Display
  },
  data() {
    return {
      products: [],
      refs: [],
      loading: null,
      completed: false
    };
  },
  mounted() {
    const input = this.$refs.input;
    input.onchange = () => {
      const file = input.files[0];
      if (file) {
        this.products = [];
        this.loading = {
          done: 0,
          total: null
        };
        file
          .text()
          .then(async data => {
            const options = { header: true, skipEmptyLines: true };
            const parsed = Papa.parse(data, options);
            const products = parsed.data;

            if (
              (parsed.errors && parsed.errors.length) ||
              products.some(product => {
                return (
                  !product.sku ||
                  !product.naam ||
                  !product.prijs ||
                  !product.link
                );
              })
            ) {
              throw new Error();
            }

            while (products.length) {
              this.products.push(
                ...products.splice(0, 10).map(product => {
                  const price = product.prijs;
                  const suffix = ',00'.slice(price.split(/^\d*/)[1].length);
                  return {
                    sku: product.sku,
                    url: product.link,
                    name: product.naam,
                    price: price + suffix
                  };
                })
              );
              this.loading = {
                done: this.products.length,
                total: this.products.length + products.length
              };
              await sleep(500);
            }

            if ( this.loading.done ) this.completed = true
          })
          .catch(() => {
            alert('Error parsing the file');
            this.loading = null;
          });
      }
    };
  },
  beforeUpdate() {
    this.refs = [];
  },
  methods: {
    downloadAll() {
      const zip = new JSZip();
      this.refs.forEach(ref => {
        const qrcode = ref.toText();
        const fileName = ref.product.sku + ' - ' + ref.product.name + '.svg';
        zip.file(fileName, qrcode);
      });
      zip.generateAsync({ type: 'blob' }).then(content => {
        saveAs(content, 'QR codes.zip');
      });
    },
    setRef(el) {
      if (el) this.refs.push(el);
    }
  }
};
</script>
<style>
body {
  margin: 0;
  background-color: #f5f5f5;
}

svg {
  display: block;
}

* {
  box-sizing: border-box;
}

.app {
  min-height: 100vh;
  max-width: 1200px;
  padding: 24px 16px;
  margin: 0 auto;
  font-family: 'Roboto', sans-serif;
}

.dropzone {
  padding: 48px 0;
  display: block;
  text-align: center;
  background-color: #ffffff;
  box-shadow: rgba(0, 0, 0, 0.1) 0 1px 3px, rgba(0, 0, 0, 0.06) 0 1px 2px;
  border-radius: 4px;
  font-weight: 500;
  cursor: pointer;
  color: #1a73e8;
  font-size: 28px;
  letter-spacing: 1px;
}

.dropzone:hover {
  background-color: #fafafa;
}

.dropzone.disabled {
  pointer-events: none;
}

.list {
  display: grid;
  grid-template-columns: repeat(auto-fit, 160px);
  align-items: start;
  margin-top: 24px;
  gap: 16px;
}

.card {
  box-shadow: rgba(0, 0, 0, 0.1) 0 1px 3px, rgba(0, 0, 0, 0.06) 0 1px 2px;
  cursor: pointer;
  user-select: none;
}

.loading {
  margin-top: 24px;
}

.btn {
  display: block;
  margin-top: 24px;
  background-color: #1a73e8;
  padding: 12px 20px;
  border-radius: 4px;
  border: none;
  font: inherit;
  font-size: 14px;
  font-weight: 500;
  text-transform: uppercase;
  box-shadow: rgba(0, 0, 0, 0.1) 0 1px 3px, rgba(0, 0, 0, 0.06) 0 1px 2px;
  letter-spacing: 1px;
  color: #ffffff;
  cursor: pointer;
  user-select: none;
  outline: none;
}
</style>
