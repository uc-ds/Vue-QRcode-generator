<template>
  <svg ref="svg" v-bind="attrs" v-on:click="download">
    <defs v-html="font"></defs>
    <g v-html="'<style></style>'"></g>
    <rect width="100%" height="100%" fill="white" />
    <g v-html="qrcode"></g>
    <g v-html="logo"></g>
  </svg>
</template>
<script>
import SvgText from 'svg-text';
import QRCode from 'qrcode-svg';
import { saveAs } from 'file-saver';

export default {
  props: {
    product: Object
  },
  data() {
    return {
      attrs: {
        width: 160,
        height: 320,
        xmlns: 'http://www.w3.org/2000/svg'
      },
      qrcode: '',
      font: `
        <style>
          @import url('https://fonts.googleapis.com/css2?family=Lato&display=swap');
        </style>
      `,
      logo: `
        <rect width="36" height="36" x="62" y="62" fill="white" />
        <svg width="32" height="32" x="64" y="64" viewBox="0 0 512 512" fill="none" xmlns="http://www.w3.org/2000/svg">
          <rect width="512" height="512" fill="#4669ca" />
          <path fill-rule="evenodd" clip-rule="evenodd" d="M258.5 102C343.284 102 412 170.716 412 255.5C412 340.284 343.262 409 258.5 409H105V102H258.5ZM250.686 187.123H181.732V256.077C181.732 294.175 212.61 325.036 250.686 325.036L252.059 325.022C289.501 324.29 319.619 293.699 319.64 256.077C319.64 217.984 288.762 187.123 250.686 187.123Z" fill="white" />
        </svg>
        </svg>
      `
    };
  },
  mounted() {
    // generate QR code
    const qrcode = new QRCode({
      content: this.product.url,
      height: 160,
      width: 160,
      join: true,
      ecl: 'H',
      color: '#4669ca',
      container: 'g'
    });
    this.qrcode = qrcode.svg();

    // generate name
    const name = new SvgText({
      text: this.product.name,
      element: this.$refs.svg,
      styleElement: this.$refs.svg.querySelector('style'),
      x: 80,
      y: 160,
      width: 160,
      padding: '0 12',
      align: 'center',
      style: {
        fill: '#4669ca',
        fontFamily: 'Lato',
        fontWeight: 'bold'
      },
      attrs: {
        'font-size': '10px'
      }
    });

    // generate price
    const price = new SvgText({
      text: `€ ${this.product.price}`,
      element: this.$refs.svg,
      styleElement: this.$refs.svg.querySelector('style'),
      x: 80,
      y:
        160 +
        name.bounds.height +
        name.lineHeight / 2 +
        (name.lines === 1 ? name.lineHeight : 0),
      width: 160,
      padding: '0 12',
      align: 'center',
      style: {
        fill: '#4669ca',
        fontFamily: 'Lato'
      },
      attrs: {
        'font-size': '14px'
      }
    });

    const height = price.bounds.y + price.bounds.height + name.lineHeight / 2;
    this.attrs.height = height;
  },
  methods: {
    toText() {
      return this.$refs.svg.outerHTML;
    },
    download() {
      const fileName = this.product.sku + ' - ' + this.product.name + '.svg';
      saveAs(new Blob([this.toText()], { type: 'image/svg+xml' }), fileName);
    }
  }
};
</script>
