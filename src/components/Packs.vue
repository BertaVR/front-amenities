<template>
    <div class="packs">
        <form class="add-item">
            <div class="inputsCaracteristicas">
                <input type="text" name="nombre" placeholder="Nombre" required />
                <input class="items" type="text" name="item1" placeholder="Item 1" required />
            </div>

            <input type="submit" class="btn btn-primary" value="Añadir Pack" />
            <input type="reset" class="btn btn-danger" value="Reset" />
        </form>
        <button id="mostrarInventario" type="button" class="btn btn-primary">Mostrar inventario</button>
        <button
            id="ocultarInventario"
            type="button"
            class="btn btn-danger"
            hidden
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
.items-incluidos {
    font-weight: bold;
    color: rgb(49, 0, 141);
}
.inputsCaracteristicas {
    display: flex;
    flex-direction: column;
    margin-left: 30%;
    margin-right: 30%;
}
.inputsCaracteristicas > input {
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
window.onload = function () {

    const serverip = '127.0.0.1:3000'

    const mostrarButton = document.getElementById('mostrarInventario');
    mostrarButton.addEventListener("click", inventarioPacks);

    function inventarioPacks() {


        var miHeaders = new Headers();

        var miInit = {
            method: 'GET',
            headers: miHeaders,
            mode: 'cors',
            // cambiarlo a force-cache => carga del disco
            cache: 'default'
        };


        fetch(`http://${serverip}/packs`, miInit)
            .then((response) => {
                if (response.ok) {
                    console.log("Response Status:", response.status);
                    console.log("Reponse statuts text:", response.statusText);
                    response.json().then((json) => {
                        logPacks(json);
                        console.log(json)
                    })
                    changeInventButton();

                } else {
                    console.log("Response Status:", response.status);
                    console.log("Reponse statuts text:", response.statusText);
                }
            })
            .catch((error) => {
                console.log(error.message);
            });
    }




    function logPacks(packs) {
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
                                         <button id="pack${i}" type="button" class="btn btn-danger borrarPack">Borrar pack</button>

                        </li>
                        `;
        }).join('');
        activarOpcionBorrar();
    }

    function activarOpcionBorrar() {
        document.querySelectorAll('.borrarPack').forEach(b => {
            b.addEventListener('click', borrarPack);

        });
    }


    function borrarPack(e) {
        let id_pack = e.target.id
        let nombre_pack = document.querySelector(`#${id_pack} span`).innerText
        if (confirm(`Borrar ${nombre_pack}`)) {

            fetch(`http://${serverip}/packs/${nombre_pack}`, {
                method: 'DELETE',
                headers: {
                    'Content-Type': 'application/json'
                }
            })
                .then((response) => {
                    if (response.ok) {
                        console.log("Response OK Status:", response.status);
                        console.log("Reponse OK status text:", response.statusText);
                        inventarioPacks();
                    } else {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                    }
                })
                .catch((error) => {
                    console.log(error.message);
                });
            //ACTUALIZAR LISTA ITEMS
        }
    }
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
    function changeInventButton() {
        document.getElementById('ocultarInventario').hidden = !document.getElementById('ocultarInventario').hidden
        document.getElementById('mostrarInventario').hidden = !document.getElementById('ocultarInventario').hidden


    }



    const ocultarButton = document.getElementById('ocultarInventario');
    ocultarButton.addEventListener("click", ocultarInventario);

    function ocultarInventario() {
        document.getElementById("packList").innerHTML = '';
        changeInventButton();

    }
}
</script>