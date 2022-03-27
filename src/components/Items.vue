<template>
    <div class="items">
        <form class="add-item">
            <div class="inputsCaracteristicas">
                <input type="text" name="nombre" placeholder="Nombre" required />
                <input type="number" step="any" name="precio" placeholder="Precio" required />
                <input type="number" name="calidad" placeholder="Calidad" required />
                <input type="text" name="material" placeholder="Material" required />

                <input type="number" name="demanda" placeholder="Demanda" required />
                <input type="number" name="stock" placeholder="Stock" required />
            </div>
            <input type="submit" class="btn btn-primary" value="Añadir Item" />
            <input type="reset" class="btn btn-danger" value="Reset" />
        </form>
        <button id="mostrarInventario" type="button" class="btn btn-primary">Mostrar inventario</button>
        <button
            id="ocultarInventario"
            type="button"
            class="btn btn-danger"
            hidden
        >Ocultar inventario</button>
        <ul id="itemList" class="list-group">
            <li>Aqui los items...</li>
        </ul>
    </div>
</template>
<style>
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
</style>
<script>
//import $ from 'jquery'

window.onload = function () {

    const serverip = '127.0.0.1:3000'

    const inventButton = document.getElementById('mostrarInventario');
    inventButton.addEventListener("click", inventario);

    function inventario() {


        var miHeaders = new Headers();

        var miInit = {
            method: 'GET',
            headers: miHeaders,
            mode: 'cors',
            // cambiarlo a force-cache => carga del disco
            cache: 'default'
        };


        fetch(`http://${serverip}/items`, miInit)
            .then((response) => {
                if (response.ok) {
                    console.log("Response Status:", response.status);
                    console.log("Reponse statuts text:", response.statusText);
                    response.json().then((json) => {
                        logItems(json)
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




    function logItems(items) {
        const itemList = document.querySelector('#itemList');
        itemList.innerHTML = items.map((item, i) => {
            return `
                        <li  class="list-group-item">
                            <p id="item${i}"><span class="nombre item${i}"><b>${item.nombre}</b></span>
                                            precio:  ${item.precio}  
                                            calidad: ${item.calidad}  
                                            material:  ${item.material}
                                            demanda: ${item.demanda}  
                                            stock: ${item.stock}  


                                            </p>
                        </li>
                        `;
        }).join('');
    }

    function changeInventButton() {
        document.getElementById('ocultarInventario').hidden = !document.getElementById('ocultarInventario').hidden
        document.getElementById('mostrarInventario').hidden = !document.getElementById('ocultarInventario').hidden


    }



    const ocultarButton = document.getElementById('ocultarInventario');
    ocultarButton.addEventListener("click", ocultarInventario);

    function ocultarInventario() {
        document.getElementById("itemList").innerHTML = '';
        changeInventButton();

    }

    let formulario = document.querySelector('.add-item');
    formulario.addEventListener('submit', addItem);


    function addItem(e) {
        e.preventDefault();
        // elementos del formulario en un array-like object:
        // form.elements 
        // pag 396 libro rhino

        let data = {
            nombre: this.elements.nombre.value,
            precio: this.elements.precio.value,
            calidad: this.elements.calidad.value,
            material: (this.elements.material.value).toLowerCase(),
            demanda: this.elements.demanda.value,
            stock: this.elements.stock.value,



        };

        fetch(`http://${serverip}/items/add`, {
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
                } else {
                    console.log("Response Status:", response.status);
                    console.log("Reponse statuts text:", response.statusText);
                }
            })
            .catch((error) => {
                console.log(error.message);
            });
    }


}
</script>