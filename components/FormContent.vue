<template>
    <section id="formContent">
            <img src="../assets/images/resplogo.svg" id="respLogo" alt="logo">
            <div class="successful" ref="success">
                <img src="../assets/images/greenCheck.svg" alt="">
                You have signed in successfully
            </div>

            <h1>Sign In to WisdomCircle</h1>
            <div>
                <p>Don't have an account?</p>
                <a href="" id="signup" @click.prevent="">Sign Up</a>
            </div>
            
            <form @submit.prevent="onSubmit">

                <div id="inputFields">
                    
                    <input type="text" placeholder="Email or Mobile Number" @blur="validateEmailOrNumber" ref="firstInput" v-model="username" id="usernameField"/>
                    <p v-show="firstInputError" ref="errorMessage" class="errorMessage"></p>
                    
                    <div id="passwordContainer">
                        <input type="password" ref="passwordField" placeholder="Password" id="passwordField" v-model="password"/>
    
                        <img v-show="!passwordError && eyeToggle" @click="togglePWVisibility" src="../assets/images/eyeGreyStrike.svg" id="eyeGreyStrike" class="eye"  alt="togglePasswordVisibility">
                        <img v-show="!passwordError && !eyeToggle" @click="togglePWVisibility" src="../assets/images/eyeGrey.svg" id="eyeGrey" class="eye"  alt="togglePasswordVisibility">
                        <img v-show="passwordError && eyeToggle" @click="togglePWVisibility" src="../assets/images/eyeRedStrike.svg" id="eyeRedStrike"  class="eye" alt="togglePasswordVisibility">
                        <img v-show="passwordError && !eyeToggle" @click="togglePWVisibility" src="../assets/images/eyeRed.svg" id="eyeRed" class="eye" alt="togglePasswordVisibility">
                    </div>
    
    
                    <div id="pwDetails"> 
                        <p v-show="passwordError" ref="pwErrorMessage" class="errorMessage" >Sorry! Password entered is incorrect</p>
                        <a href="" id="forgotPW" @click.prevent="">Forgot password</a>
                    </div>

                </div>

                
                <button>Sign In</button>
            </form>
    </section>
</template>


<script>

export default {
    components: {

    },

    data() {
        return {
            firstInputError: false,
            inputValid: false,
            username: '',
            password: '',
            usernameErrorDB: false,
            passwordError: false,
            inputEmail: false,
            inputNumber: false,
            eyeToggle: false
        }
    },

    methods: {
        validateEmailOrNumber(event) {
            const val = event.target.value 
            
            // if the field is empty
            if (!val) {
                this.firstInputError = false
                this.inputValid = false
                return 
            }
            
            // if the field is not a valid email
            const emailRegex = /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}$/i;
            const mobileRegex = /^([0-9]{10})$/

            if (!emailRegex.test(val) && !mobileRegex.test(val)) {
                this.$refs.errorMessage.textContent = 'Please enter a valid email or phone number'

                this.styleNode(this.$refs.firstInput, "error")
                this.firstInputError = true
                this.inputValid = false
            }
            
            else{
                if(emailRegex.test(val)){
                    this.inputEmail = true
                    this.inputNumber = false
                }
                else{
                    this.inputNumber = true
                    this.inputEmail = false
                }
                this.inputValid = true

                if (!this.usernameErrorDB){
                    this.styleNode(this.$refs.firstInput, "default")
                    this.firstInputError = false
                }
            }
        },

        togglePWVisibility(e){
            this.eyeToggle = !this.eyeToggle
            if (this.eyeToggle){
                this.$refs.passwordField.setAttribute("type", "text")
            }
            else {
                this.$refs.passwordField.setAttribute("type", "password")
            }
        },
        onSubmit(){
            if (this.inputValid){
                console.log(this.username, this.password, 'has been submitted')

                let userDetails = {}
                userDetails["username"] = this.username
                userDetails["password"] = this.password

                fetch('http://localhost:4000',{
                    method: 'POST',
                    body: JSON.stringify(userDetails),
                    headers: {
                        'Content-Type': 'application/json'
                    }
                })
                .then(res => res.json())
                .then(data => {
                    if(data.usernameError){
                        this.firstInputError = true
                        this.usernameErrorDB = true

                        let textContent = ''
                        if(this.inputEmail){
                            textContent = 'email'
                        }
                        else{
                            textContent = 'mobile number'
                        }
                        this.$refs.errorMessage.textContent = `Sorry! This ${textContent} is not registered`
                        this.styleNode(this.$refs.firstInput, "error")
                    }
                    else if(data.passwordError){
                        this.firstInputError = false
                        this.usernameErrorDB = false
                        this.passwordError = true

                        this.styleNode(this.$refs.firstInput, "default")
                        this.styleNode(this.$refs.passwordField, "error")
                    }
                    else{
                        this.firstInputError = false
                        this.usernameErrorDB = false
                        this.passwordError = false
                        
                        this.$refs.success.setAttribute('id', 'active')

                        setTimeout(()=> {
                            this.$refs.success.removeAttribute('id')
                        }, 3000)

                        this.styleNode(this.$refs.firstInput, "default")
                        this.styleNode(this.$refs.passwordField, "default")
                    }
                })  
                .catch(err => console.log(err))
            }
            else{
                console.log("Please enter valid credentials")
            }
        },
        styleNode(node, status){
            let color
            let borderColor
            let errorColour = '#D92D20'
            if (status=="error"){
                color = errorColour
                borderColor = errorColour
            }
            else if (status=="default"){
                color = 'black'
                borderColor = 'rgba(0, 0, 0, 0.152)'
            }
            node.style.color = color
            node.style.borderColor = borderColor
        }
    }
}
</script>
