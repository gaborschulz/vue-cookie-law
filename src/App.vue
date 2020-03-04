<template>
  <div id="app">
    <select v-model="theme">
      <option value="base">Base</option>
      <option value="blood-orange">Blood Orange</option>
      <option value="dark-lime">Dark Lime</option>
      <option value="royal">Royal</option>
      <option value="base--rounded">Base Round</option>
      <option value="blood-orange--rounded">Blood Orange Round</option>
      <option value="dark-lime--rounded">Dark Lime Round</option>
      <option value="royal--rounded">Royal Round</option>
    </select>
    <cookie-law :theme="theme" 
                position="top" 
                transition-name="fade" 
                :buttonDecline="true"
                buttonDeclineText="Accept Selected"
                buttonText="Accept All"
                :cookieTypes="cookieTypes"
                buttonLink="https://example.com"
                :buttonLinkNewTab="true"
                :timeout="10000">
      <template v-slot:message>
        <span style="display: block; text-align: left; font-size: 0.8em;">
          <h3>Cookie Consent</h3>
          <p>
            This website uses cookies/local storage for the purpose of tracking your acceptance of the cookie policy. 
            This item is <strong>essential</strong> to comply with the law and maintain an acceptable browsing experience. 
            We would have to ask you for your consent after every navigation step without this. 
            This cookie remains valid for 1 month, after which we'll ask you again for your consent.
          </p>
          <p>
            We may also use some <strong>measurement</strong> cookies described in our cookie policy to analyze the way you use our site and improve your experience. 
            These are not required for using the site and you can choose to accept or reject these as you like. 
            You can freely give, reject, or revoke your consent without incurring major limitations at any time.
          </p>
          <p>
            You accept the use of cookies or other identifiers by closing or dismissing this notice, by clicking a link or button or by continuing to browse otherwise.
          </p>
        </span>
      </template>
    </cookie-law>
    <!-- <cookie-law :theme="theme">
      <div slot-scope="props" class="container">

        <button class="skew" @click="props.accept"><span>I accept</span></button>
        <p>
          Lorem ipsum, dolor sit amet consectetur adipisicing elit. Ex, hic?
        </p>

        <button class="skew" @click="props.close"><span>Close</span></button>
      </div>

    </cookie-law> -->
  </div>
</template>

<script>
import CookieLaw from './components/CookieLaw'

export default {
  name: 'app',
  components: {
    CookieLaw
  },
  data () {
    return {
      theme: 'website--rounded',
      cookieTypes: [
        { id: 'essential', required: true, label: 'Essential' },
        { id: 'measurement', required: false, label: 'Measurement' }
      ]
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
  margin-top: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
}

select {
  width: 400px;
  height: 40px;
  font-size: 18px;
}

.skew {
  background: rebeccapurple;
  color: #fff;
  font-size: 0.875rem;
  font-weight: 700;
  text-transform: uppercase;
  border-radius: 8px 0;
  transform: skew(-20deg);
  padding: 0.2rem 1rem;
}

.skew span {
  display: inline-block;
  transform: skew(20deg);
}

.container {
  width: 100%;
  text-align: center;
}
</style>
