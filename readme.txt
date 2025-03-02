Estoy trabajando en un proyecto en react, tengo el siguiente componente:
Componente LISTAFAMILIAPROFESIONALES
a. Devolverá una etiqueta <div>
b. Mostrará la lista de familias profesionales obtenidas del ENDPOINT de
familias profesionales
c. Se elegirá un componente seleccionable para cada una de las familias
profesionales (Ver componente de partida para botones de filtros)
d. Este componente devolverá un array con la lista de familias seleccionadas

El cual funciona y esta formado por el get:
export function getFamiliasProfesionales() {

    const url = 'http://marcapersonalfp.test/api/v1/familias_profesionales';

    return fetch(url, {
        method: 'GET',

    })
        .then(res => res.json())
        .then(data => {
            return data.map (familia => ({
                nombre: familia.nombre,
                codigo: familia.codigo,
                id: familia.id
            }));
        })
        .catch((error) => {
            console.error('Error al obtener Familias Profesionales:', error);
            return [];
        });
}
el use
import { useEffect, useState } from 'react';
import { getFamiliasProfesionales } from '../../servicios/getFamiliasProfesionales';

const useFamiliasProfesionales = () => {
    // Estado para almacenar las familias profesionales
    const [familiasProfesionales, setFamiliasProfesionales] = useState([]);
    

    // Función para obtener las familias profesionales
    function obtenerFamiliasProfesionales () {
        getFamiliasProfesionales().then(data => {
            setFamiliasProfesionales(data);
        }
        );
    }

    // Para ejecutar una sola vez
    useEffect(obtenerFamiliasProfesionales, []);

    return {familiasProfesionales};


}

export default useFamiliasProfesionales;

Y el componente:
import React, { useState } from 'react';
import useFamiliasProfesionales from '../hooks/useFamiliasProfesionales';
import './ListaFamiliasProfesionales.css';

const ListaFamiliasProfesionales = ({ onSeleccionarFamilias }) => {
    const { familiasProfesionales } = useFamiliasProfesionales();
    const [buscando, setBuscando] = useState(false);
    const [familiasSeleccionadas, setFamiliasSeleccionadas] = useState([]);

    function mostrarFamilias() {
        setBuscando(!buscando);
    }

    function cambiarFamiliaSeleccionada(familiaId) {
        const nuevasFamiliasSeleccionadas = familiasSeleccionadas.includes(familiaId)
            ? familiasSeleccionadas.filter(id => id !== familiaId)
            : [...familiasSeleccionadas, familiaId];

        setFamiliasSeleccionadas(nuevasFamiliasSeleccionadas);
        onSeleccionarFamilias(nuevasFamiliasSeleccionadas);
    }

    return (
        <div className="container">
            <div className="pb-3">
                <button className="boton-filtrar" onClick={mostrarFamilias}>
                    Filtrar por familia profesional {buscando ? '▲' : '▼'}
                </button>
                {buscando && (
                    <div className="mt-2">
                        {Array.isArray(familiasProfesionales) && familiasProfesionales.map((familia, index) => (
                            <div className="customCheckBoxHolder" key={familia.id}>
                                <input
                                    type="checkbox"
                                    id={cCB${index}}
                                    className="customCheckBoxInput"
                                    checked={familiasSeleccionadas.includes(familia.id)}
                                    onChange={() => cambiarFamiliaSeleccionada(familia.id)}
                                />
                                <label htmlFor={cCB${index}} className="customCheckBoxWrapper">
                                    <div className="customCheckBox">
                                        <div className="inner">{familia.nombre}</div>
                                    </div>
                                </label>
                            </div>
                        ))}
                    </div>
                )}
            </div>
        </div>
    );
};

export default ListaFamiliasProfesionales;

Que deberia filtrar dependiendo de los que selecciones.

Luego, necesito que mirando la imagen, el correcto funcionamiento de los componentes C y D
Su definicion es:
Componente RESULULTADOSBUSQUEDAPROYECTOS
a. Devolverá una etiqueta <div className="row">
b. Mostrará inicialmente la lista de todos los proyectos existentes que los
obtendremos del ENDPOINT de proyectos.
c. Conforme vayamos aplicando filtros, los filtros se aplicarán a dicha lista.
d. Si como resultado de los filtros no se muestra ningún proyecto se debe
indicar con un mensaje.
e. Cada proyecto se mostrará con el componente PROYECTOMINCARD
Componente PROYECTOMINCARD
a. Devolverá una etiqueta <div>
b. Por cada proyecto se mostrará
a. Nombre del proyecto
b. Imagen. De momento poner esta url:
https://source.unsplash.com/random/200x200/?proyecto
c. Alumnos participantes en el proyecto
d. Tutor (De momento poner el docente-id)
e. Ciclos formativos asociados al proyecto. Se mostrarán los códigos
del ciclo (codCiclo) y como tooltip su nombre completo.


Y el codigo que yo tengo es:
getProyectos:
export function getProyectos() {
    return fetch("http://marcapersonalfp.test/api/v1/proyectos", {
      method: "GET",
    }).then((response) => {
      const data = response.json();
      return data;
    });
  }

useProyectos:
import { useEffect, useState } from "react";
import { getProyectos } from "../../servicios/getProyectos";
const useProyectos = () => {

    const [proyectos, setProyectos] = useState([]);
    const [buscando, setBuscando] = useState(false)

    function obtenerProyectos () {

        setBuscando(true);

        getProyectos().then(datos => {
            
            setProyectos(datos);
            setBuscando(false);
        })
    }

    useEffect(obtenerProyectos, []);
     console.log({proyectos})
    return {proyectos, buscando}
}
export default useProyectos;

useProyectosFiltrados:
import { useEffect, useState } from 'react';
import useProyectos from './useProyectos';

const useProyectosFiltrados = () => {
    const { proyectos, buscando } = useProyectos();
    const [listaProyectos, setListaProyectos] = useState([]);
    const [listaProyectosFiltrados, setListaProyectosFiltrados] = useState([]);

    useEffect(() => {
        if (!buscando) {
            setListaProyectos(proyectos);
            setListaProyectosFiltrados(proyectos);
        }
    }, [proyectos, buscando]);

    const filtrarLista = (familiasId) => {
        if (familiasId.length === 0) {
            setListaProyectosFiltrados(listaProyectos);
        } else {
            const proyectosFiltrados = listaProyectos.filter(proyecto =>
                proyecto.familias.some(familia => familiasId.includes(familia.id))
            );
            setListaProyectosFiltrados(proyectosFiltrados);
        }
    };

    return { listaProyectosFiltrados, filtrarLista };
};

export default useProyectosFiltrados;


ResultadoBusquedaProyectos:
import React, { useState, useEffect } from 'react';
import ProyectoMinCard from "../ProyectoMinCard/ProyectoMinCard";

const ResultadosBusquedaProyectos = ({ listaProyectosFiltrados }) => {
    const [mostrar, setMostrar] = useState(true);
    const [proyectos, setProyectos] = useState([]);

    useEffect(() => {
        if (listaProyectosFiltrados) {
            setProyectos(listaProyectosFiltrados);
        }
    }, [listaProyectosFiltrados]);

    function mostrandoProyectos() {
        setMostrar(!mostrar);
    }

    return (
        <div className="container mt-3 busqueda-proyectos">
            <div className="border p-3">
                <h5 className="fw-bold">Resultados</h5>
                <button className="boton-filtrar" onClick={mostrandoProyectos}>
                    Proyectos {mostrar ? '▲' : '▼'}
                </button>
                {mostrar && proyectos.length > 0 ? (
                    proyectos.map((proyecto) => (
                        <ProyectoMinCard key={proyecto.id} proyecto={proyecto} />
                    ))
                ) : (
                    mostrar && <p>No se encontraron proyectos para las familias seleccionadas.</p>
                )}
            </div>
        </div>
    );
};

export default ResultadosBusquedaProyectos;

Y finalmente la pagina donde incluyo todo:

import Cabecera from "../../componentes/Cabecera/Cabecera";
import ListaFamiliasProfesionales from "../../componentes/ListaFamiliasProfesionales/ListaFamiliasProfesionales";
import MenuEmpresa from "../../componentes/MenuEmpresa/MenuEmpresa";
import { useState, useEffect } from 'react';
import useProyectosFiltrados from "../../componentes/hooks/useProyectosFiltrados";

const BusquedaProyectos = (props) => {
    const [idioma, setIdioma] = useState(props.idioma);
    const { listaProyectosFiltrados, filtrarLista } = useProyectosFiltrados();
    const [familiasSeleccionadas, setFamiliasSeleccionadas] = useState([]);

    function manejarSeleccion(idiomaSeleccionado) {
        setIdioma(idiomaSeleccionado);
        mandarSeleccion(idiomaSeleccionado);
    }

    function mandarSeleccion(idiomaSeleccionado) {
        props.manejarSeleccion(idiomaSeleccionado);
    }

    const handleSeleccionarFamilias = (familiasSeleccionadas) => {
        setFamiliasSeleccionadas(familiasSeleccionadas);
    };

    useEffect(() => {
        const familiasIds = familiasSeleccionadas.map(familia => familia.id);
        filtrarLista(familiasIds);
    }, [familiasSeleccionadas, filtrarLista]);

    return (
        <>
            <Cabecera manejarSeleccion={manejarSeleccion} />
            <h4>Búsqueda de Proyectos</h4>
            <MenuEmpresa />
            <div>
          <ListaFamiliasProfesionales
            filtrarLista={filtrarLista}
          ></ListaFamiliasProfesionales>
        </div>

        </>
    );
};

export default BusquedaProyectos;


El problema es que ahora mismo ni me funciona el componente C ni D, y que antes de aplicar ningun filtro deberian presentarse todos los proyectos en las Cards de proyectos y ya ir filtrando segun seleccione en familias_profesionales, necesito que me arregles el codigo, me crees o borres los componentes, gets o use que veas necesario para que cumpla las condiciones establecidas
