<template>
	<section class="py-5" id="contact">
		<h3 class="text-center pb-5">Contact</h3>
		<div class="container">
			<div class="row">
				<div class="col-sm-12 col-md-6">
					<iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d15427.53017329402!2d121.076033!3d14.8318488!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x3397af29a0c7ff6b%3A0xd6bdaa93458419c6!2sCenterville%20Brgy%20Sto%20Cristo!5e0!3m2!1sen!2sph!4v1779381540057!5m2!1sen!2sph" width="100%" height="450" style="border:0;" allowfullscreen="" loading="lazy" referrerpolicy="no-referrer-when-downgrade"></iframe>
				</div>
				
				<div class="col-sm-12 col-md-6 border border-3 p-4 rounded">
				  <div class="form-card">
				    <form @submit.prevent="submitForm">
				      <div class="mb-3">
				        <label class="form-label fw-semibold small">Full Name</label>
				        <input type="text" v-model="name" class="form-control" placeholder="First Name, M.I., Last Name">
				      </div>
				      
				      <div class="mb-3">
				        <label class="form-label fw-semibold small">Email Address</label>
				        <input type="email" v-model="email" class="form-control" placeholder="Email">
				      </div>
				      
				      <div class="mb-4">
				        <label class="form-label fw-semibold small">Message</label>
				        <textarea class="form-control" rows="5" placeholder="Message" v-model="message"></textarea>
				      </div>
				      
				      <div class="d-flex justify-content-between align-items-center">
				        <div class="d-flex gap-2">
				          <a href="#"><img src="/images/linkedin.svg"></a>
				          <a href="#"><img src="/images/indeed.svg"></a>
				          <a href="#"><img src="/images/kalibrr.svg"></a>
				          <a href="#"><img src="/images/github.svg"></a>
				        </div>
				        <div class="d-flex justify-content-end mt-2">
                            <div ref="recaptchaContainer"></div>
                        </div>
				        <button type="submit" class="btn btn-submit btn-highlight" :disabled="isLoading">{{isLoading ? "Sending..." : "Submit"}}</button>
				      </div>
				    </form>
				  </div>
				</div>
			
			</div>
		</div>
	</section>
</template>

<script setup>
	
	import { ref, onMounted, onBeforeUnmount } from 'vue';
	import { Notyf } from 'notyf';
	import 'notyf/notyf.min.css';

	const notyf = new Notyf();
	const WEB3FORMS_ACCESS_KEY = "a86adb1f-5d77-4773-b179-fc7f6317e024";
	const subject = "New message from Portfolio Contact Form";

	const name = ref("");
	const email = ref("");
	const message = ref("");
	const isLoading = ref(false);

	
	//FROM MS. NIKKIE
	// const submitForm = async() => {
	// 	isLoading.value = true;
	// 	try{
	// 		const response = await fetch("https://api.web3forms.com/submit", {
	// 			method: "POST",
	// 			headers: {
	// 				"Content-Type": "application/json",
	// 				Accept: "application/json",
	// 			},
	// 			body: JSON.stringify({
	// 				access_key: WEB3FORMS_ACCESS_KEY,
	// 				subject: subject,
	// 				name: name.value,
	// 				email: email.value,
	// 				message: message.value,
	// 			})
	// 		});
	// 		const result = await response.json();

	// 		if(result.success){
	// 			console.log(result);
	// 			isLoading.value = false;
	// 			notyf.success("Message sent!")
	// 		}
	// 	}catch(error){
	// 		console.log(error);
	// 		isLoading.value = false;
	// 		notyf.error("Failed to send message.")
	// 	}
	// }

	const submitForm = async () => {

		if(!recaptchaToken.value){
			notyf.error("Please verify that you are not a robot.")
			return;
		}

	  isLoading.value = true;
	  try {
	    const response = await fetch("https://api.web3forms.com/submit", {
	      method: "POST",
	      headers: {
	        "Content-Type": "application/json",
	        Accept: "application/json",
	      },
	      body: JSON.stringify({
	        access_key: WEB3FORMS_ACCESS_KEY,
	        subject: subject,
	        name: name.value,
	        email: email.value,
	        message: message.value,
	      }),
	    });
	    const result = await response.json();

	    if (result.success) {
	      notyf.success("Message sent!");
	      name.value = "";
	      email.value = "";
	      message.value = "";
	    } else {
	      notyf.error(result.message || "Failed to send message.");
	    }
	  } catch (error) {
	    console.error(error);
	    notyf.error("Failed to send message. Please try again.");
	  } finally {
	    isLoading.value = false;
	    resetRecaptcha();
	  }
	};


	const SITE_KEY = '6Ld9RYItAAAAAK8PP6xdM8b3VrjSsimY8jLH_yr0';  // Replace with your site key

	const recaptchaContainer = ref(null);
	const recaptchaWidgetId = ref(null);
	const recaptchaToken = ref('');

	// Callback called by reCAPTCHA when successful
	function onRecaptchaSuccess(token) {
	  recaptchaToken.value = token;
	}

	// Callback when expired
	function onRecaptchaExpired() {
	  recaptchaToken.value = '';
	}

	// Function to render the reCAPTCHA widget
	function renderRecaptcha() {
	  if (!window.grecaptcha) {
	    console.error('reCAPTCHA not loaded');
	    return;
	  }

	  recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
	    sitekey: SITE_KEY,
	    size: 'normal', // or 'compact'
	    callback: onRecaptchaSuccess,
	    'expired-callback': onRecaptchaExpired,
	  });
	}

	// Function to reset reCAPTCHA 
	function resetRecaptcha() {
	  if (recaptchaWidgetId.value !== null) {
	    window.grecaptcha.reset(recaptchaWidgetId.value);
	    recaptchaToken.value = '';
	  }
	}



	onMounted(() => {
	  // This code waits for the Google reCAPTCHA library to load, then renders the reCAPTCHA widget using onMounted hook. 
	  // The widget is rendered with grecaptcha.render(), which requires a sitekey. 
	  // Callback functions handle success and expiration events. 
	  // reCAPTCHA is reset upon form submission to clear the token.
	  const interval = setInterval(() => {
	    if (window.grecaptcha && window.grecaptcha.render) {
	      renderRecaptcha();
	      clearInterval(interval);
	    }
	  }, 100);

	  onBeforeUnmount(() => {
	    clearInterval(interval);
	  });
	});

</script>