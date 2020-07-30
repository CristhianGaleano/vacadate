<template>
<!-- https://firebase.google.com/docs/auth/web/manage-users -->
<!-- https://desarrolloweb.com/articulos/introduccion-firebase-storage.html -->
<!-- v-layout: en bootstrap es el container -->
    <v-layout justify-center align-center>



        <!-- La column -->
        <v-flex xs12 sm8 md6 lg5 xl4 mr-5  align-center>
            <!-- la estructura  -->
            <v-card>
                <v-toolbar color="infoPublicidad">
                    <v-toolbar-title>
                        Información de Interés
                    </v-toolbar-title>
                </v-toolbar>
                <!-- <v-card-text>
                    <v-text-field autofocus v-model="email" label="email"></v-text-field>
                    <v-text-field @keyup.enter="ingresar" v-model="password" label="Password" type="password"></v-text-field>
                </v-card-text> -->
                <v-card-text>
                    <v-layout justify-center>
                       <div class="row">
                           <div class="col-md-12">
                               <a href="https://www.ucp.edu.co/pregrado/">Oferta de Pregrados</a>
                           </div>
                           <div class="col-md-12">
                               <a href="https://www.ucp.edu.co/posgrado/">Oferta de Posgrados</a>
                           </div>
                       </div>
                    </v-layout>
                </v-card-text>
            </v-card>
        </v-flex>





        <!-- La column -->
        <v-flex xs12 sm8 md6 lg5 xl4>
            <!-- la estructura  -->
            <v-card>
                <v-toolbar color="primary" dark>
                    <v-toolbar-title>
                        Conéctate con uno de nuestros asesores
                    </v-toolbar-title>
                </v-toolbar>
                <!-- <v-card-text>
                    <v-text-field autofocus v-model="email" label="email"></v-text-field>
                    <v-text-field @keyup.enter="ingresar" v-model="password" label="Password" type="password"></v-text-field>
                </v-card-text> -->
                <v-card-text>
                    <v-layout justify-end>
                        <v-btn @click="ingresar" color="infoPublicidad">Inicia sesión con Google</v-btn>
                    </v-layout>
                </v-card-text>
            </v-card>
        </v-flex>


    </v-layout>
    
</template>


<script>

import {auth,provider,db,storage} from '@/firebase'
// import {auth, db} from '@/firebase'

export default {
    props: {
       
    },
    watch: {
       
    },
    data () {
        return {
            email: '',
            password: ''
        }
    },
    methods: {
       async ingresar () {

           
          
        try {
            
            await auth.signInWithPopup(provider)
            if(auth.currentUser){

                console.log('Usuario registrado');
                let uidCurrentUser = auth.currentUser.uid
                let nameCurrentUser = auth.currentUser.displayName
                let emailCurrentUser = auth.currentUser.email
                let photoCurrentUser = auth.currentUser.photoURL
                let lastSignInTime = auth.currentUser.metadata.lastSignInTime
                console.log('UID: ' + uidCurrentUser);
                console.log('Name: ' + nameCurrentUser);
                console.log('email: ' + emailCurrentUser);
                console.log('photo: ' + photoCurrentUser);
                console.log('lastSignInTime: ' + lastSignInTime);

                let usuario = {
                    foto: photoCurrentUser,
                    uid: uidCurrentUser,
                    nombre: nameCurrentUser,
                    lastSignInTime: lastSignInTime,
                    rol: 'user'
                }


                await db.collection('usuarios')
                        .doc(uidCurrentUser)
                        .set(usuario)
                        .then(function() {
                            console.log("Document successfully written!");   
                        }).catch(function () {
                            console.log("Document not success");
                        });

                  //  acces for document on BD
                let doc = await db.collection('usuarios')
                                .doc(uidCurrentUser)
                                .get()
                                console.log(doc);
                if(doc.exists){
                    console.log('Exits');
                    let usuario = doc.data()
                    this.$emit('onIngresar', usuario)
                }else{
                    this.enviarNotificacion('No se encontró la información del usuario', 'Error')
                }

                storage.ref('usuarios/'+uidCurrentUser+'/photo.jpg')
                let task = storage.put(photoCurrentUser)
                console.log(task);
            }
               
        } catch (error) {
            console.log(error)
        }           
            },
        
        enviarNotificacion (mensaje, color) {
            this.$emit('onNotificacion', { mensaje, color })
        },
        executeLogout () {
            auth.signOut()
                .then(function (){
                    console.log("logout succesful");
                })
                .catch(function (error){
                    console.log(error);
                })
        },
        // sesionUser () {
        //      console.log('create method');
        //     auth.onAuthStateChanged(user=>{
        //     if(user){
        //         // console.log(user);
        //         this.usuario = user
        //         console.log('CD-User signed: ',user);
        //     }
        //     })
        // }
    },

}
</script>