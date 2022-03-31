<template>
    <div class="packs">
        <form class="add-item">
            <div id="inputsCaracteristicas">
                <input
                    class="form-control"
                    type="text"
                    ref="nombre"
                    name="nombre"
                    placeholder="Nombre"
                    required
                />
                <div id="itemList" ref="itemList">
                    <input
                        class="items form-control"
                        ref="item"
                        type="text"
                        placeholder="Item"
                        required
                    />
                </div>
            </div>
            <div class="inputs-botones">
                <div class="boton-superior">
                    <button @click="addItem" type="button" class="btn btn-success">
                        <svg
                            xmlns="http://www.w3.org/2000/svg"
                            width="16"
                            height="16"
                            fill="currentColor"
                            class="bi bi-plus"
                            viewBox="0 0 16 16"
                        >
                            <path
                                d="M8 4a.5.5 0 0 1 .5.5v3h3a.5.5 0 0 1 0 1h-3v3a.5.5 0 0 1-1 0v-3h-3a.5.5 0 0 1 0-1h3v-3A.5.5 0 0 1 8 4z"
                            />
                        </svg> Añadir Item
                    </button>
                </div>
                <div class="botones-inferiores">
                    <input
                        type="submit"
                        class="btn btn-primary add-pack"
                        @click="addPack"
                        value="Añadir Pack"
                    />
                    <input type="reset" class="btn btn-danger" value="Reset" />
                </div>
            </div>
        </form>
        <div class="mensajes">
            <span hidden class="mensaje" id="exito">Item creado</span>
            <span hidden class="mensaje" id="error"></span>
        </div>
        <button
            id="mostrarInventario"
            @click="inventarioPacks"
            type="button"
            class="btn btn-primary"
        >Mostrar inventario</button>
        <button
            id="ocultarInventario"
            type="button"
            class="btn btn-danger"
            hidden
            @click="ocultarInventario"
        >Ocultar inventario</button>
        <ul id="packList" class="list-group">
            <li></li>
        </ul>
    </div>
</template>
<style>
ul {
    list-style: none;
}

.inputs-botones {
    display: flex;
    flex-direction: column;
    margin: 2px;
    padding: 2px;
}
.inputs-botones > div {
    margin: 5px;
}

.botones-inferiores > input {
    margin: 2px;
}

.items-incluidos {
    font-weight: bold;
    color: rgb(49, 0, 141);
}
#inputsCaracteristicas {
    display: flex;
    flex-direction: column;
    margin-left: 30%;
    margin-right: 30%;
}
#inputsCaracteristicas > div {
    display: flex;
    flex-direction: column;
}
#inputsCaracteristicas > div > input {
    margin: 4px;
}
#inputsCaracteristicas > input {
    margin: 4px;
}
form {
    margin-bottom: 30px;
}
button {
    margin-top: 4px;
}
</style>

<script>

export default {

    data() {
        return {
            serverip: '127.0.0.1:3000',
        }



    },
    created() {
        this.manageItemInputs

    },
    methods: {
        inventarioPacks() {


            var miHeaders = new Headers();

            var miInit = {
                method: 'GET',
                headers: miHeaders,
                mode: 'cors',
                // cambiarlo a force-cache => carga del disco
                cache: 'default'
            };


            fetch(`http://${this.serverip}/packs`, miInit)
                .then((response) => {
                    if (response.ok) {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                        response.json().then((json) => {
                            this.logPacks(json);
                            console.log(json)
                        })
                        this.changeInventButton();

                    } else {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                    }
                })
                .catch((error) => {
                    console.log(error.message);
                });
        },




        logPacks(packs) {
            const packList = document.querySelector('#packList');
            packList.innerHTML = packs.map((pack, i) => {
                return `
                        <li  class="list-group-item">
                            <p id="pack${i}"><span class="nombre pack${i}"><b>${pack.nombre}</b></span>
                                            precio:  ${pack.precio}  
                                            stock:  ${pack.stock}
                                            <ul>
                                        <lh class="items-incluidos"> Items incluidos:</lh>

                                            ${pack.items.map((item) => {
                    //TODO:  Refactor
                    return `
                          <li  class="item_in_pack">
                              <p id="item"><span class="nombre item"><b>${item.nombre}</b></span>
                                                                        material:  ${item.material}  
                                                                        calidad:  ${item.calidad}  
                                                                        precio:  ${item.precio}  
                                                                        demanda:  ${item.demanda}  

                                              </p>
                          </li>
                          `;
                }).join('')}
                                            </ul>
                                            </p>
                                         <button id="pack${i}" @click="borrarPack" type="button" class="btn btn-danger borrarPack">Borrar pack</button>

                        </li>
                        `;
            }).join('');
            this.activarOpcionBorrar();
        },

        activarOpcionBorrar() { //refactor vue
            document.querySelectorAll('.borrarPack').forEach(b => {
                b.addEventListener('click', this.borrarPack);

            });
        },
        borrarPack(e) {
            let id_pack = e.target.id
            let nombre_pack = document.querySelector(`#${id_pack} span`).innerText
            if (confirm(`Borrar ${nombre_pack}`)) {

                fetch(`http://${this.serverip}/packs/${nombre_pack}`, {
                    method: 'DELETE',
                    headers: {
                        'Content-Type': 'application/json'
                    }
                })
                    .then((response) => {
                        if (response.ok) {
                            console.log("Response OK Status:", response.status);
                            console.log("Item borrado:", response.statusText);
                            this.inventarioPacks();
                        } else {
                            console.log("Response Status:", response.status);
                            console.log("Reponse statuts text:", response.statusText);
                        }
                    })
                    .catch((error) => {
                        console.log(error.message);
                    });
            }
        },

        changeInventButton() {
            document.getElementById('ocultarInventario').hidden = !document.getElementById('ocultarInventario').hidden
            document.getElementById('mostrarInventario').hidden = !document.getElementById('ocultarInventario').hidden


        },
        ocultarInventario() {
            document.getElementById("packList").innerHTML = '';
            this.changeInventButton();

        },
   
        addItem() {
            const addItemInput = document.createElement('input');
            addItemInput.placeholder = 'Item';
            addItemInput.type = 'text';
            const contenedor = document.getElementById('itemList');
            contenedor.appendChild(addItemInput);
            addItemInput.classList.add('items', 'form-control');

        },

        getItemsInCurrentPack() {
            let items = this.$refs.itemList.children
            let arrayItems = [];
            for (var i = 0; i < items.length; i++) {
                if (items[i].value)
                    arrayItems.push(items[i].value);
            }
            return arrayItems;
        },
        addPack(e) {
            e.preventDefault();

            let data = {
                nombre: this.$refs.nombre.value,
                items: this.getItemsInCurrentPack(),




            };

            fetch(`http://${this.serverip}/packs/add`, {
                method: 'POST',
                body: JSON.stringify(data),
                headers: {
                    'Content-Type': 'application/json'
                }
            })
                .then((response) => {
                    if (response.ok) {
                        console.log("Response OK Status:", response.status);
                        console.log("Reponse OK status text:", response.statusText);
                        document.getElementById('error').hidden = true;
                        document.getElementById('exito').hidden = false; 0
                    } else {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                        document.getElementById('exito').hidden = true;
                        document.getElementById('error').innerHTML = `Ha habido un error: ${response.statusText} <br> Recuerde añadir sólo items que existan actualmente en el inventario de items `;
                        document.getElementById('error').hidden = false;
                    }
                })
                .catch((error) => {
                    console.log(error.message);
                });
        }


    }
}








/* function */




/* TENGO UN PROBLEMA CON LAS PROMESAS
 
function renderItems(items) {
      items.map((item, i) => {
          return `
                      <li  class="item_in_pack">
                          <p id="item${i}"><span class="nombre item${i}"><b>${item.nombre}</b></span>
  
 
 
                                          </p>
                      </li>
                      `;
      }).join('');
 
  }*/










</script>