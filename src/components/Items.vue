<template>
    <div class="items">
        <button id="inventario">inventario</button>

        <ul id="itemList">
            <li>Aqui los items...</li>
        </ul>
        <form class="add-item">
            <input type="text" name="nombre" placeholder="Nombre" required />
            <input type="number" step="any" name="precio" placeholder="Precio" required />
            <input type="number" name="calidad" placeholder="Calidad" required />
            <input type="text" name="material" placeholder="Material" required />

            <input type="number" name="demanda" placeholder="Demanda" required />
            <input type="number" name="stock" placeholder="Stock" required />
            <input type="submit" value="Añadir Item" />
            <input type="reset" value="Reset" />
        </form>
    </div>
</template>

<script>
window.onload = function () {

    const serverip = '127.0.0.1:3000'
    
    const inventButton = document.querySelector('#inventario');
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
                    response.json().then((json) =>{ logItems(json)
                    console.log(json)})
                    
                } else {
                    console.log("Response Status:", response.status);
                    console.log("Reponse statuts text:", response.statusText);
                }
            })
            .catch((error) => {
                console.log(error.message);
            });
    }

    // intentar cachear con la cabecera mirando network de chrome

   function logItems(items) {
        const itemList = document.querySelector('#itemList');
        itemList.innerHTML = items.map((item, i) => {
            return `
                        <li>
                            <p id="item${i}"> ${item.nombre}
                                           ${item.precio}  
                                            ${item.calidad}  
                                            ${item.material}
                                             ${item.demanda}  
                                             ${item.stock}  


                                            </p>
                        </li>
                        `;
        }).join('');
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