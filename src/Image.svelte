<script>
  export let file, alt;
</script>

<img {alt} height="400" width="400"
  src={URL.createObjectURL(file)}>
<hr>

<style>
  img {
      margin: 0 auto;
      display: block;
  }
  hr {
      margin: 10px;
  }
</style>