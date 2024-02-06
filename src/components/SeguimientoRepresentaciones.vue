<template>
    <div class="d-flex align-center flex-column">
        <v-card width="auto" elevation="1" variant="flat" :color="color">
          <v-card-title class="mt-6 mb-6 text-center text-wrap">
            <v-icon left>
                mdi-checkbox-marked-outline
            </v-icon>
            Seguimiento de la(s) Sesion(es) de los Consejos Distritales
          </v-card-title>
          <!--<v-card-subtitle class="text-center mb-6">Ingresar datos</v-card-subtitle>-->
        </v-card>
    </div>    
    <v-container fluid class="mt-6">
      <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
        <v-col col="12" xs="12" sm="12" md="12" lg="12">
          <v-card class="mx-auto" max-width="600" variant="tonal" :color="color">
            <v-card-item>
              <div>
                <div class="text-overline mb-1">
                  Sesiones
                </div>
                <div class="text-h6 mb-1">
                  <v-icon lef > mdi-ballot-outline </v-icon>
                  Selecciona la sesión a celebrar
                </div>
                <div class="text-caption">Sólo estarán las sesiones que se dieron de alta en el día</div>
              </div>
            </v-card-item>
            <v-card-text>
              <v-select
                clearable
                label="Seleccionar una opción"
                :items="select_sesion"
                item-title="name" item-value="value"
                :color="color"
                variant="outlined"
                v-model="sesion"
                @update:modelValue="llenaTabla"
              ></v-select>
            </v-card-text>
          </v-card>
        </v-col>
        <v-col col="12" xs="12" sm="12" md="6" lg="6" v-if="sesion != null">  

        </v-col> 
        <v-col col="12" xs="12" sm="12" md="11" lg="11" v-if="sesion != null">
          <v-card variant="outlined" :color="color" elevation="3">
            <v-data-table v-model:page="page" :headers="headers" :items="desserts" items-per-page="20" hide-default-footer v-if="sesion != null"
                no-data-text="No hay datos disponibles" :color = "color" show-expand item-value = "tipo_sesion"  item-key = "idSesion">
                <template v-slot:bottom>
                  <div class="text-center pt-2">
                    <v-pagination
                      v-model="page"
                      :length="pageCount"
                    ></v-pagination>
                  </div>
                </template>
                <template v-slot:[`item.cat_integrantes`]="{ item }">
                  <template v-if="item.completada == 1">
                    <v-tooltip text="La asistencia de cierre ya ha sido tomada"  location="bottom">
                      <template v-slot:activator="{ props }">
                        <v-icon v-bind="props" color="red">mdi-block-helper</v-icon>
                      </template>
                    </v-tooltip>
                  </template>
                  <v-btn v-else color="#512077" variant="text" @click="abreDialogIntegrantes(item)" icon="mdi mdi-account-box-multiple-outline"></v-btn>
                </template>
                <template v-slot:[`item.inicio_sesion`]="{ item }">
                  <template v-if="item.completada == 1">
                    <v-tooltip text="La asistencia de cierre ya ha sido tomada"  location="bottom">
                      <template v-slot:activator="{ props }">
                        <v-icon v-bind="props" color="red">mdi-block-helper</v-icon>
                      </template>
                    </v-tooltip>
                  </template>
                  <v-btn v-else-if="item.completada == null" color="#D32F2F" variant="text" @click="inicio_sesion(item)" icon="mdi mdi-clock-time-eight-outline" ></v-btn>
                  <v-btn v-else-if="item.completada == 0" color="light-blue" variant="text" @click="consulta_inicio_sesion(item)" icon="mdi-clock-edit-outline"></v-btn>
                </template>
                <template v-slot:[`item.fin_sesion`]="{ item }">
                  <template v-if="item.completada == 1">
                    <v-tooltip text="La asistencia de cierre ya ha sido tomada"  location="bottom">
                      <template v-slot:activator="{ props }">
                        <v-icon v-bind="props" color="red">mdi-block-helper</v-icon>
                      </template>
                    </v-tooltip>
                  </template>
                  <template v-else-if="item.completada == null">
                    <v-tooltip text="Asistencia de inicio aún no tomada"  location="bottom">
                      <template v-slot:activator="{ props }">
                        <v-icon v-bind="props" color="light-blue">mdi-timer-sand</v-icon>
                      </template>
                    </v-tooltip>
                  </template>
                  <!-- <v-btn v-else-if="item.completada == 0" color="#D32F2F" variant="text" @click="fin_sesion(item)" icon="mdi mdi-clock-time-eight-outline"></v-btn> -->
                  <v-btn v-else-if="item.contadosVotacion < item.numVotacionRequired && item.completada == 0" color="#D32F2F" variant="text" @click="snackBarFaltantes" icon="mdi-clock-time-eight-outline"></v-btn>
                  <v-btn v-else-if="item.contadosVotacion == item.numVotacionRequired && item.completada == 0" color="#D32F2F" variant="text" @click="fin_sesion(item)" icon="mdi-clock-time-eight-outline"></v-btn>
                </template>
                <template v-slot:[`item.seg_orden_dia`]="{ item }">
                  <template v-if="item.completada == 1">
                    <v-tooltip text="La asistencia de cierre ya ha sido tomada"  location="bottom">
                      <template v-slot:activator="{ props }">
                        <v-icon v-bind="props" color="red">mdi-block-helper</v-icon>
                      </template>
                    </v-tooltip>
                  </template>
                  <v-btn v-else color="#7986CB" variant="text" @click="seg_orden_dia(item)" icon="mdi mdi-clipboard-text-clock-outline"></v-btn>
                </template>
                <template v-slot:[`item.estado_sesion`]="{ item }">
                  <template v-if="item.completada == 1">
                    <v-tooltip text="La asistencia de cierre ya ha sido tomada"  location="bottom">
                      <template v-slot:activator="{ props }">
                        <v-icon v-bind="props" color="red">mdi-block-helper</v-icon>
                      </template>
                    </v-tooltip>
                  </template>
                  <v-btn v-else color="#4DD0E1" variant="text" @click="estado_sesion(item)" icon="mdi mdi-checkbox-outline"></v-btn>
                </template>
                <template v-slot:expanded-row="{ columns, item }">
                  <tr>
                    <td :colspan="columns.length" class="text-left">
                      <v-divider class="mb-2"></v-divider>
                      Reportes {{ item.reportes }}
                      <v-btn color="#4DB6AC" variant="text" @click="reportesDialogs(item.idSesion)" icon="mdi mdi-book-arrow-down-outline"></v-btn>
                      <v-divider class="my-2"></v-divider>
                      Documentos de la sesión {{ item.acreditacion }} 
                      <v-btn color="#81C784" variant="text" @click="doc_sesion(item.idSesion, item.documentoSesion)" icon="mdi mdi-file-document-outline"></v-btn>
                      <v-divider class="my-2"></v-divider>
                      Acta {{ item.comision }}
                      <v-btn color="#90A4AE" variant="text" @click="acta(item.idSesion, item.actaSesion)" icon="mdi mdi-file-sign"></v-btn>
                      <v-divider class="my-2"></v-divider>
                      Minuta {{ item.reportes }}
                      <v-btn color="#E64A19" variant="text" @click="minuta(item.idSesion, item.minutaSesion)" icon="mdi mdi-file-document-edit-outline"></v-btn>
                      <v-divider class="mt-2"></v-divider>
                    </td>
                  </tr>
                </template>      
            </v-data-table>      
          </v-card>
        </v-col>
      </v-row>
    </v-container>
    <v-dialog v-model="dialogIntegrantesConsejo" content-class="rounded-xl elevation-12" scrollable width="80%" :retain-focus="false">
      <v-card class="rounded-xl">
        <v-container class="ma-0 pa-0" fluid style="height: 110px">
          <v-img cover src="../img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
              <v-card title="Actualización de Integrantes del Consejo" class="text-white" color="rgba(0, 0, 0, 0)" variant="flat">
                  <!-- <v-avatar :color="color" size="60">
                      <v-icon large>
                          mdi-account-group
                      </v-icon>
                  </v-avatar> -->
              </v-card>
          </v-img>
        </v-container>
        <v-toolbar :color="color">
          <v-btn icon @click="dialogIntegrantesConsejo = false">
            <v-icon left >mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title>
            <v-tabs v-model="tab2" :bg-color="color" centered stacked  align-tabs="center">
              <v-tab value="tab-1">
                  <v-icon>mdi-account-heart-outline</v-icon>
                  Consejo Distrital
              </v-tab>

              <v-tab value="tab-2">
                  <v-icon>mdi-human-capacity-decrease</v-icon>
                  Partidos Políticos
              </v-tab>

              <v-tab value="tab-3">
                  <v-icon>mdi-human-male-female</v-icon>
                  Candidaturas sin Partido
              </v-tab>
            </v-tabs>
          </v-toolbar-title>
        </v-toolbar>
        <v-card-text class="rounded-xl" >
          <v-card class="justify-center align-center text-center" variant="flat" >
            <v-window v-model="tab2">
              <v-window-item value="tab-1">
                <v-card >
                  <v-card-text>
                    <v-form ref="formConsejo">
                        <template v-for="integrante in dessertsIntegrantes" :key="integrante.idIntegrante">
                            <v-list-item :title="integrante['cargo']" >
                                <template v-slot:prepend>
                                    <v-avatar :color="color" size="50">
                                        <strong>{{ integrante['siglas'] }}</strong>
                                    </v-avatar>
                                </template>
                                <v-container fluid>
                                    <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                                        <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                            <v-text-field label="Nombre" v-model="integrante['nombre']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                        </v-col>
                                        <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                            <v-text-field label="Apellido Paterno" v-model="integrante['primerApellido']" :maxlength="150" :counter="150" :rules="rulesObligatory" :color="color" variant="outlined"></v-text-field>
                                        </v-col>
                                        <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                            <v-text-field label="Apellido Materno" v-model="integrante['segundoApellido']" :maxlength="150" :counter="150" :rules="rulesObligatory" :color="color" variant="outlined"></v-text-field>
                                        </v-col>
                                        <v-col col="12" xs="12" sm="12" md="12" lg="12" ><v-divider ></v-divider></v-col>
                                    </v-row>
                                </v-container>
                            </v-list-item>
                        </template>
                    </v-form>
                    <v-container fluid>
                        <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                            <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                <v-btn variant="tonal" size="large" color="red" @click="dialogIntegrantesConsejo = false"> Cerrar </v-btn>
                            </v-col>
                            <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                <v-tooltip text="Este cambio será aplicado a todos los distritos de la sesión"  location="top">
                                    <template v-slot:activator="{ props }">
                                        <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" :color="color" block size="large" v-bind="props" @click="validaConsejo">
                                            <v-icon left>
                                            mdi-content-save-move-outline
                                            </v-icon> Guardar
                                        </v-btn>
                                    </template>
                                </v-tooltip>
                            </v-col>
                        </v-row>
                    </v-container>
                  </v-card-text>
                </v-card>
              </v-window-item>
              <v-window-item value="tab-2">
                      <v-card>
                          <v-card-text>
                              <!-- <v-virtual-scroll :items="itemsScroll" height="450" item-height="80"> -->
                                  <v-form ref="formPartidos">
                                      <template v-for="partido in dessertsPartidos" :key="partido.idIntegrantePartido">
                                          <v-list-item :title="partido['nombrePartido'] + ' (' + partido['siglas'] + ')'" >
                                              <template v-slot:prepend>
                                                  <v-img class="mx-auto" contain :lazy-src="'src/img/logos/'+ partido['logoPartido']" width="70" :src="'src/img/logos/'+ partido['logoPartido']"></v-img> 
                                              </template>
                                              <v-container fluid>
                                                  <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center" v-if="partido['PS'] == 'P'">
                                                      <v-col col="12" xs="12" sm="12" md="12" lg="12"> Propietario</v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Nombre" v-model="partido['nombre']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Apellido Paterno" v-model="partido['primerApellido']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Apellido Materno" v-model="partido['segundoApellido']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                  </v-row>
                                                  <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center" v-if="partido['PS'] == 'S'">
                                                      <v-col col="12" xs="12" sm="12" md="12" lg="12"> Suplente</v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Nombre" v-model="partido['nombre']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Apellido Paterno" v-model="partido['primerApellido']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Apellido Materno" v-model="partido['segundoApellido']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="12" lg="12"><v-divider></v-divider></v-col>
                                                  </v-row>
                                              </v-container>
                                          </v-list-item>
                                      </template>
                                  </v-form>
                              <!-- </v-virtual-scroll> -->
                              <v-container fluid>
                                  <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center my-0 py-0">
                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                          <v-btn variant="tonal" size="large" color="red" @click="dialogIntegrantesConsejo = false"> Cerrar </v-btn>
                                      </v-col>
                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                          <v-tooltip text="Este cambio será aplicado a todos los distritos de la sesión"  location="top">
                                              <template v-slot:activator="{ props }">
                                                  <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" :color="color" block size="large" v-bind="props" @click="validaPartidos">
                                                      <v-icon left>
                                                      mdi-content-save-move-outline
                                                      </v-icon> Guardar
                                                  </v-btn>
                                              </template>
                                          </v-tooltip>
                                      </v-col>
                                  </v-row>
                              </v-container>
                          </v-card-text>
                      </v-card>
                  </v-window-item>
                  <v-window-item  value="tab-3">
                      <v-card>
                          <v-card-text>
                              <!-- <v-virtual-scroll  :items="itemsScroll" height="450" item-height="80"> -->
                                  <v-form ref="formIndependientes">
                                      <template  v-for="independiente in dessertsIndependientes" :key="independiente.idIntegrantePartido">
                                          <v-list-item :title="independiente['nombrePartido'] + ' (' + independiente['siglas'] + ')'" >
                                              <template v-slot:prepend>
                                                  <v-avatar :color="color" size="50">
                                                      <strong>{{ independiente['siglas'] }}</strong>
                                                  </v-avatar>
                                              </template>
                                              <v-container fluid>
                                                  <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center" v-if="independiente['PS'] == 'P'">
                                                      <v-col col="12" xs="12" sm="12" md="12" lg="12"> Propietario</v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Nombre" v-model="independiente['nombre']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Apellido Paterno" v-model="independiente['primerApellido']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Apellido Materno" v-model="independiente['segundoApellido']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                  </v-row>
                                                  <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center" v-if="independiente['PS'] == 'S'">
                                                      <v-col col="12" xs="12" sm="12" md="12" lg="12"> Suplente</v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Nombre" v-model="independiente['nombre']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Apellido Paterno" v-model="independiente['primerApellido']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                                          <v-text-field label="Apellido Materno" v-model="independiente['segundoApellido']" :color="color" :maxlength="150" :counter="150" :rules="rulesObligatory" variant="outlined"></v-text-field>
                                                      </v-col>
                                                      <v-col col="12" xs="12" sm="12" md="12" lg="12"><v-divider></v-divider></v-col>
                                                  </v-row>
                                              </v-container>
                                          </v-list-item>
                                      </template>
                                  </v-form>
                              <!-- </v-virtual-scroll> -->
                              <v-container fluid>
                                  <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center my-0 py-0">
                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                          <v-btn variant="tonal" size="large" color="red" @click="dialogIntegrantesConsejo = false"> Cerrar </v-btn>
                                      </v-col>
                                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                                          <v-tooltip text="Este cambio será aplicado a todos los distritos de la sesión"  location="top">
                                              <template v-slot:activator="{ props }">
                                                  <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" :color="color" block size="large" v-bind="props" @click="validaIndependientes">
                                                      <v-icon left>
                                                      mdi-content-save-move-outline
                                                      </v-icon> Guardar
                                                  </v-btn>
                                              </template>
                                          </v-tooltip>
                                      </v-col>
                                  </v-row>
                              </v-container>
                          </v-card-text>
                      </v-card>
                  </v-window-item>
              </v-window>
          </v-card>
        </v-card-text>
        <!-- <v-card-actions>
          <v-container fluid>
            <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                <v-col col="12" xs="12" sm="12" md="4" lg="4">
                    <v-btn variant="tonal" size="large" color="red" @click="dialogIntegrantesConsejo = false"> Cerrar </v-btn>
                </v-col>
                <v-col col="12" xs="12" sm="12" md="4" lg="4">
                    <v-tooltip text="Este cambio será aplicado a todos los distritos de la sesión"  location="top">
                        <template v-slot:activator="{ props }">
                            <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" :color="color" block size="large" v-bind="props" @click="validaConsejo">
                                <v-icon left>
                                mdi-content-save-move-outline
                                </v-icon> Guardar
                            </v-btn>
                        </template>
                    </v-tooltip>
                </v-col>
            </v-row>
          </v-container>
        </v-card-actions> -->
      </v-card>
    </v-dialog>
    <!--  Inicia inicio de sesión   -->
    <v-dialog v-model="dialogInicioSesion" content-class="rounded-xl elevation-12" scrollable width="80%" :retain-focus="false">
      <v-card class="rounded-xl">
        <v-img height="100" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
          <v-card title="Inicio de Sesión" color="rgba(0, 0, 0, 0)" class="text-white" variant="flat">
            <!-- <v-card-text>
              Verificar primero el catálogo de integrantes antes de iniciar sesión.
            </v-card-text> -->
          </v-card>
        </v-img>
        <v-toolbar :color="color">
          <v-btn icon @click="dialogInicioSesion = false">
            <v-icon left >mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="text-center">
            Verificar primero el catálogo de integrantes antes de iniciar sesión.
          </v-toolbar-title>
        </v-toolbar>
        <v-card-text class="justify-center align-center text-center">
          <v-container fluid>
            <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
              <!-- <v-col col="12" xs="12" md="4" lg="4" class="text-center">
                Fecha de inicio
              </v-col> -->
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="d-flex text-center justify-center align-content-center ">
                <v-card class="mt-0 pt-0" width="70%" variant="tonal" :color="color">
                  <v-card-item title="Datos de la sesión">
                    <template v-slot:subtitle>
                      <v-icon icon="mdi-information" size="20" :color="color" class="me-1 pb-1"></v-icon>
                      Estos son de referencia de la sesión a celebrar
                    </template>
                  </v-card-item>

                  <v-card-text class="pt-0">
                    <v-row align="center" no-gutters>
                      <v-col class="text-h5" cols="10" >
                        <p>Fecha programada: {{ form1.fecha }}</p>
                        <p>Hora programada: {{ form1.horaProgramada }}</p>
                      </v-col>
                      <v-col cols="2" class="text-center">
                        <v-icon
                          :color="color"
                          icon="mdi-calendar-clock"
                          size="50"
                        ></v-icon>
                      </v-col>
                    </v-row>
                  </v-card-text>
                </v-card>
                
                <!-- <v-dialog ref="dialogRegCat" v-model="modal" persistent width="350">  -->
                  <!-- <template v-slot:activator="{ props }"> -->
                    <!-- <v-text-field readonly :color="color" v-model="form1.fecha" prepend-icon="mdi-calendar" label="Indicar la fecha de inicio" 
                    variant="outlined" :error-messages="fechaErrors" @change="v$.form1.fecha.$touch()" @blur="v$.form1.fecha.$touch()" ref="fecha"></v-text-field> -->
                  <!-- </template>
                  <v-card>
                    <v-card-text class="justify-center align-center text-center ma-0 pa-0">
                      <div class="d-flex justify-center">
                        <v-locale-provider locale="es">
                          <v-date-picker :color="color" title="Selecciona una fecha" header="Calendario" width="350" min="2023-01-01" max="2025-01-01" mode="date" @update:modelValue="updateDate"></v-date-picker>
                        </v-locale-provider>
                      </div>
                    </v-card-text>
                    <v-card-actions class="justify-center align-center text-center">
                      <v-btn variant="text" :color="color" @click="modal = false">
                        Aceptar
                      </v-btn>
                    </v-card-actions>
                  </v-card>
                  
                </v-dialog> -->
              </v-col> 
              <v-col col="0" xs="0" sm="0" md="0" lg="0"></v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center">Hora de inicio</v-col>
              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                <v-text-field :color="color" v-model="form1.hora" label="Hora" :maxlength="5"
                    variant="outlined" :error-messages="horaErrors" @change="v$.form1.hora.$touch()" @blur="v$.form1.hora.$touch()" ref="hora">
                  <template #prepend>
                    <v-dialog ref="dialogRegH" v-model="modal2"  persistent width="350px" overlay-opacity=".1">
                      <template v-slot:activator="{ props }">
                        <v-btn  size="x-large" v-bind="props" density="compact" :color="color"><v-icon>mdi-clock-time-eight-outline</v-icon></v-btn>
                      </template>
                      <v-card class="align-center mx-8">
                        <time-picker :color="color" v-model="form1.hora" automatic></time-picker>
                        <v-card-actions>
                          <v-btn variant="text" :color="color" @click="modal2 = false">Aceptar</v-btn>
                        </v-card-actions>
                      </v-card>
                    </v-dialog>
                  </template>  
                </v-text-field>
                <!-- <v-dialog ref="dialogRegH" v-model="modal2"  persistent width="350px" overlay-opacity=".1">
                  <template v-slot:activator="{ props }">
                    <v-text-field readonly :color="color" v-model="form1.hora" prepend-icon="mdi-clock-time-eight-outline" label="Hora" 
                    variant="outlined" v-bind="props" :error-messages="horaErrors" @change="v$.form1.hora.$touch()" @blur="v$.form1.hora.$touch()" ref="hora"></v-text-field>
                  </template>
                  <v-card class="align-center mx-8">
                    <time-picker :color="color" v-model="form1.hora" automatic></time-picker>
                    <v-card-actions>
                      <v-btn variant="text" :color="color" @click="modal2 = false">Aceptar</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog> -->
              </v-col> 
              <v-col col="0" xs="0" sm="0" md="0" lg="0"></v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center">
                Domicilio de la Sede Distrital:
              </v-col>
              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                <v-textarea type="text" :color="color" v-model="form1.domicilio" :counter="2000" min="1" label="Domicilio de la Sede Distrital" variant="outlined" 
                :error-messages="domicilioErrors" @change="v$.form1.domicilio.$touch()" @blur="v$.form1.domicilio.$touch()" ref="nombre" maxlength ="2000"></v-textarea>
              </v-col> 
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                Asistencia a inicio de sesión de Consejo
              </v-col>
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                <v-table fixed-header widht="400">
                  <thead>
                    <tr>
                      <th class="text-center">
                        Nombre
                      </th>
                      <th class="text-center">
                        Cargo
                      </th>
                      <th class="text-center">
                        Asistencia
                      </th>                        
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="integrantes in dessertsIntegrantes" :key="integrantes['idIntegrante']">
                      <td class="text-center">{{ integrantes['nombre'] }}</td>
                      <td class="text-center">{{ integrantes['siglas'] }}</td>
                      <td class="text-center">
                        <v-checkbox class="d-inline-flex" :color="color" v-model="integrantes['asistencia']"></v-checkbox>
                      </td>
                    </tr>
                  </tbody>
                </v-table>
              </v-col>
              <v-form ref="formIntegrantesInicio">
                <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                  Asistencia al Inicio de la Sesión Representantes de Partidos Políticos
                </v-col>
                <v-container fluid>
                  <v-row class="d-flex text-center justify-center align-content-center ">
                    <template v-for="integrantes in dessertsPartidos" :key="integrantes.idIntegrante">
                      <v-col cols="auto" xs="3" sm="3" md="3" lg="3" class="d-flex text-center justify-center align-content-center ">
                        <v-table fixed-header>
                          <thead>
                            <tr>
                              <th class="text-center justify-center align-content-center">
                                {{ integrantes['siglasPartido'] }}
                                <v-tooltip text="Limpiar asistencia" location="top">
                                  <template v-slot:activator="{ props }">
                                    <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>  
                                  </template>
                                </v-tooltip>
                              </th>                     
                            </tr>
                          </thead>
                          <tbody >
                            <tr>
                              <td class="text-center justify-center align-content-center">
                                <v-radio-group  inline :color="color" v-model="integrantes['asistencia']" >
                                  <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                    <template v-slot:activator="{ props }">
                                      <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                    </template>
                                  </v-tooltip> -->
                                  <!-- <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                    <template v-slot:activator="{ props }">
                                      <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                    </template>
                                  </v-tooltip> -->
                                  <v-radio label="P" value="P"></v-radio>
                                  <v-radio label="S" value="S"></v-radio>
                                </v-radio-group>
                              </td>
                            </tr>
                          </tbody>
                        </v-table>
                      </v-col>
                    </template>
                  </v-row>
                </v-container>
                <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                  Asistencia a Sesión de Candidaturas Sin Partido
                </v-col>
                <v-container fluid>
                  <v-row class="d-flex text-center justify-center align-content-center ">
                    <template v-for="integrantes in dessertsIndependientes" :key="integrantes.idIntegrante">
                      <v-col cols="auto" xs="3" sm="3" md="3" lg="3">
                        <div >
                          <v-table fixed-header >
                            <thead>
                              <tr>
                                <th class="text-center">
                                  {{ integrantes['nombrePartido'] }} <br>( {{ integrantes['siglasPartido'] }} )
                                  <v-tooltip text="Limpiar asistencia" location="top">
                                    <template v-slot:activator="{ props }">
                                      <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>
                                    </template>
                                  </v-tooltip>
                                </th>    
                              </tr>
                            </thead>
                            <tbody>
                              <tr >
                                <td class="text-center justify-center align-content-center">
                                  <v-radio-group inline :color="color" v-model="integrantes['asistencia']" >
                                    <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                      <template v-slot:activator="{ props }">
                                        <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                      </template>
                                    </v-tooltip>
                                    <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                      <template v-slot:activator="{ props }">
                                        <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                      </template>
                                    </v-tooltip> -->
                                    <v-radio label="P" value="P" ></v-radio>
                                    <v-radio label="S" value="S" ></v-radio>
                                  </v-radio-group>
                                        
                                </td>
                              </tr>
                            </tbody>
                          </v-table>
                        </div>
                      </v-col>
                    </template>
                  </v-row>
                </v-container>
              </v-form>
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                Asistencia de medios de comunicación:
              </v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center justify-center align-center d-inline-flex">
                <v-checkbox :color="color" v-model="form1.prensa" >Prensa</v-checkbox>
              </v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center d-inline-flex">
                <v-checkbox :color="color" v-model="form1.radio">Radio</v-checkbox>
              </v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center d-inline-flex">
                <v-checkbox :color="color" v-model="form1.television">Televisión</v-checkbox>
              </v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center d-inline-flex">
                Observaciones del inicio de Sesión:
              </v-col>
              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                <v-textarea type="text" :color="color" v-model="form1.observaciones" :counter="2000" min="1" label="Observaciones del inicio de Sesión" variant="outlined" 
                :error-messages="observacionesErrors" @change="v$.form1.observaciones.$touch()" @blur="v$.form1.observaciones.$touch()" ref="nombre" maxlength ="2000"></v-textarea>
              </v-col>                                 
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions class="d-flex justify-center">
          <v-btn variant="tonal" size="large" color="red" @click="dialogInicioSesion = false"> Cerrar </v-btn>
          <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="validaInicioSesion"> Guardar </v-btn>
        </v-card-actions>
      </v-card>  
    </v-dialog>
    <!--  termina inicio de sesión   -->
    <!-- Inicia fin de sesión -->
    <v-dialog v-model="dialogFinSesion" content-class="rounded-xl elevation-12" scrollable max-width="80%" :retain-focus="false">
      <v-card class="rounded-xl">
        <v-img height="100" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
          <v-card title="Asistencia de fin de sesión" color="rgba(0, 0, 0, 0)" class="text-white" variant="flat">
            <!-- <v-card-text>
              Verificar primero el catálogo de integrantes antes de finalizar la sesión.
            </v-card-text> -->
          </v-card>
        </v-img>
        <v-toolbar :color="color">
          <v-btn icon @click="dialogFinSesion = false">
            <v-icon left >mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="text-center">
            Verificar primero el catálogo de integrantes antes de finalizar la sesión.
          </v-toolbar-title>
        </v-toolbar>
        <v-card-text class="justify-center align-center text-center">
          <v-container fluid>
            <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="d-flex text-center justify-center align-content-center ">
                <v-card class="mt-0 pt-0" width="70%" variant="tonal" :color="color">
                  <v-card-item title="Datos de la sesión">
                    <template v-slot:subtitle>
                      <v-icon icon="mdi-information" size="20" :color="color" class="me-1 pb-1"></v-icon>
                      Estos son de referencia de la sesión a celebrar
                    </template>
                  </v-card-item>

                  <v-card-text class="pt-0">
                    <v-row align="center" no-gutters>
                      <v-col class="text-h5" cols="10" >
                        <p>Fecha programada: {{ form1.fecha }}</p>
                      </v-col>
                      <v-col cols="2" class="text-center">
                        <v-icon
                          :color="color"
                          icon="mdi-calendar-clock"
                          size="50"
                        ></v-icon>
                      </v-col>
                    </v-row>
                  </v-card-text>
                </v-card>
              </v-col> 
              <!-- <v-col col="12" xs="12" md="4" lg="4" class="text-center">
                Fecha de inicio
              </v-col>
              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                <v-dialog ref="dialogRegCat" v-model="modal" persistent width="350"> 
                  <template v-slot:activator="{ props }">
                    <v-text-field readonly :color="color" v-model="form1.fecha" prepend-icon="mdi-calendar" label="Indicar la fecha de inicio" 
                    variant="outlined" v-bind="props" :error-messages="fechaErrors" @change="v$.form1.fecha.$touch()" @blur="v$.form1.fecha.$touch()" ref="fecha"></v-text-field>
                  </template>
                  <v-card>
                    <v-card-text class="justify-center align-center text-center ma-0 pa-0">
                      <div class="d-flex justify-center">
                        <v-locale-provider locale="es">
                          <v-date-picker :color="color" title="Selecciona una fecha" header="Calendario" width="350" min="2023-01-01" max="2025-01-01" mode="date" @update:modelValue="updateDate"></v-date-picker>
                        </v-locale-provider>
                      </div>
                    </v-card-text>
                    <v-card-actions class="justify-center align-center text-center">
                      <v-btn variant="text" :color="color" @click="modal = false">
                        Aceptar
                      </v-btn>
                    </v-card-actions>
                  </v-card>
                  
                </v-dialog>
              </v-col>  -->
              <!-- <v-col col="0" xs="0" sm="0" md="0" lg="0"></v-col> -->
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center">Hora de fin</v-col>
              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                <v-text-field :color="color" v-model="form1.hora" label="Hora" :maxlength="5" 
                    variant="outlined" :error-messages="horaErrors" @change="v$.form1.hora.$touch()" @blur="v$.form1.hora.$touch()" ref="hora">
                  <template #prepend>
                    <v-dialog ref="dialogRegH" v-model="modal2"  persistent width="350px" overlay-opacity=".1">
                      <template v-slot:activator="{ props }">
                        <v-btn  size="x-large" v-bind="props" density="compact" :color="color"><v-icon>mdi-clock-time-eight-outline</v-icon></v-btn>
                      </template>
                      <v-card class="align-center mx-8">
                        <time-picker :color="color" v-model="form1.hora" automatic></time-picker>
                        <v-card-actions>
                          <v-btn variant="text" :color="color" @click="modal2 = false">Aceptar</v-btn>
                        </v-card-actions>
                      </v-card>
                    </v-dialog>
                    <!-- <v-dialog ref="dialogRegH" v-model="modal2"  persistent width="350px" overlay-opacity=".1">
                      <template v-slot:activator="{ props }">
                        <v-btn  size="x-large" v-bind="props" density="compact" :color="color"><v-icon>mdi-clock-time-eight-outline</v-icon></v-btn>
                      </template>
                      <v-card class="align-center mx-8">
                        <time-picker :color="color" v-model="form.hora" automatic></time-picker>
                        <v-card-actions>
                          <v-btn variant="text" :color="color" @click="modal2 = false">Aceptar</v-btn>
                        </v-card-actions>
                      </v-card>
                    </v-dialog> -->
                  </template>  
                  </v-text-field>
                <!-- <v-dialog ref="dialogRegH" v-model="modal2"  persistent width="350px" overlay-opacity=".1">
                  <template v-slot:activator="{ props }">
                    <v-text-field readonly :color="color" v-model="form1.hora" prepend-icon="mdi-clock-time-eight-outline" label="Hora" 
                    variant="outlined" v-bind="props" :error-messages="horaErrors" @change="v$.form1.hora.$touch()" @blur="v$.form1.hora.$touch()" ref="hora"></v-text-field>
                  </template>
                  <v-card class="align-center mx-8">
                    <time-picker :color="color" v-model="form1.hora" automatic></time-picker>
                    <v-card-actions>
                      <v-btn variant="text" :color="color" @click="modal2 = false">Aceptar</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog> -->
              </v-col> 
              <v-col col="0" xs="0" sm="0" md="0" lg="0"></v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center">
                Domicilio de la Sede Distrital:
              </v-col>
              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                <v-textarea type="text" :color="color" v-model="form1.domicilio" :counter="2000" min="1" label="Domicilio de la Sede Distrital" variant="outlined" 
                :error-messages="domicilioErrors" @change="v$.form1.domicilio.$touch()" @blur="v$.form1.domicilio.$touch()" ref="nombre" maxlength ="2000"></v-textarea>
              </v-col> 
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                Asistencia a inicio de sesión de Consejo
              </v-col>
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                <v-table fixed-header widht="400">
                  <thead>
                    <tr>
                      <th class="text-center">
                        Nombre
                      </th>
                      <th class="text-center">
                        Cargo
                      </th>
                      <th class="text-center">
                        Asistencia
                      </th>                        
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="integrantes in dessertsIntegrantes" :key="integrantes['idIntegrante']">
                      <td class="text-center">{{ integrantes['nombre'] }}</td>
                      <td class="text-center">{{ integrantes['siglas'] }}</td>
                      <td class="text-center">
                        <v-checkbox class="d-inline-flex" :color="color" v-model="integrantes['asistencia']"></v-checkbox>
                      </td>
                    </tr>
                  </tbody>
                </v-table>
              </v-col>
              <v-form ref="formIntegrantesInicio">
                <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                  Asistencia al Inicio de la Sesión Representantes de Partidos Políticos
                </v-col>
                <v-container fluid>
                  <v-row class="d-flex text-center justify-center align-content-center ">
                    <template v-for="integrantes in dessertsPartidos" :key="integrantes.idIntegrante">
                      <v-col cols="auto" xs="3" sm="3" md="3" lg="3" class="d-flex text-center justify-center align-content-center ">
                        <v-table fixed-header>
                          <thead>
                            <tr>
                              <th class="text-center justify-center align-content-center">
                                {{ integrantes['siglasPartido'] }}
                                <v-tooltip text="Limpiar asistencia" location="top">
                                  <template v-slot:activator="{ props }">
                                    <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>
                                  </template>
                                </v-tooltip>
                              </th>                     
                            </tr>
                          </thead>
                          <tbody >
                            <tr>
                              <td class="text-center justify-center align-content-center">
                                <v-radio-group  inline :color="color" v-model="integrantes['asistencia']" >
                                  <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                    <template v-slot:activator="{ props }">
                                      <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                    </template>
                                  </v-tooltip> -->
                                  <!-- <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                    <template v-slot:activator="{ props }">
                                      <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                    </template>
                                  </v-tooltip> -->
                                  <v-radio label="P" value="P"></v-radio>
                                  <v-radio label="S" value="S"></v-radio>
                                </v-radio-group>
                              </td>
                            </tr>
                          </tbody>
                        </v-table>
                      </v-col>
                    </template>
                  </v-row>
                </v-container>
                <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                  Asistencia a Sesión de Candidaturas Sin Partido
                </v-col>
                <v-container fluid>
                  <v-row class="d-flex text-center justify-center align-content-center ">
                    <template v-for="integrantes in dessertsIndependientes" :key="integrantes.idIntegrante">
                      <v-col cols="auto" xs="3" sm="3" md="3" lg="3">
                        <div >
                          <v-table fixed-header >
                            <thead>
                              <tr>
                                <th class="text-center">
                                  {{ integrantes['nombrePartido'] }} <br>( {{ integrantes['siglasPartido'] }} )
                                  <v-tooltip text="Limpiar asistencia" location="top">
                                    <template v-slot:activator="{ props }">
                                      <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>
                                    </template>
                                  </v-tooltip>
                                </th>                     
                              </tr>
                            </thead>
                            <tbody>
                              <tr >
                                <td class="text-center justify-center align-content-center">
                                  <v-radio-group inline :color="color" v-model="integrantes['asistencia']" >
                                    <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                      <template v-slot:activator="{ props }">
                                        <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                      </template>
                                    </v-tooltip>
                                    <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                      <template v-slot:activator="{ props }">
                                        <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                      </template>
                                    </v-tooltip> -->
                                    <v-radio label="P" value="P" ></v-radio>
                                    <v-radio label="S" value="S" ></v-radio>
                                  </v-radio-group>
                                </td>
                              </tr>
                            </tbody>
                          </v-table>
                        </div>
                      </v-col>
                    </template>
                  </v-row>
                </v-container>
              </v-form>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center justify-center align-center d-inline-flex">
                <v-checkbox :color="color" v-model="form1.prensa" >Prensa</v-checkbox>
              </v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center d-inline-flex">
                <v-checkbox :color="color" v-model="form1.radio">Radio</v-checkbox>
              </v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center d-inline-flex">
                <v-checkbox :color="color" v-model="form1.television">Televisión</v-checkbox>
              </v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center">
                Observaciones del fin de Sesión:
              </v-col>
              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                <v-textarea type="text" :color="color" v-model="form1.observaciones" :counter="2000" min="1" label="Observaciones del inicio de Sesión" variant="outlined" 
                :error-messages="observacionesErrors" @change="v$.form1.observaciones.$touch()" @blur="v$.form1.observaciones.$touch()" ref="nombre" maxlength ="2000"></v-textarea>
              </v-col>                                 
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions class="d-flex justify-center">
          <v-btn variant="tonal" size="large" color="red" @click="dialogFinSesion = false"> Cerrar </v-btn>
          <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="validaFinSesion"> Guardar </v-btn>
        </v-card-actions>
      </v-card>  
    </v-dialog>
    <!-- termina Fin de sesión -->

    <!-- Inicio de orden de seguimiento  -->
    <v-dialog v-model="dialogOrdenDDia" content-class="rounded-xl elevation-12" scrollable max-width="80%" :retain-focus="false">
      <v-card >
        <v-img height="80" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
            <v-card title="Seguimiento del orden del día" class="text-white" color="rgba(0, 0, 0, 0)" variant="flat">
            </v-card>
        </v-img>
        <v-toolbar :color="color" v-if="pasoPuntos == 'window-1'">
          <v-btn icon @click="dialogOrdenDDia = false">
            <v-icon left >mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
              Puntos de orden del día para la sesión
          </v-toolbar-title>
        </v-toolbar>
        <v-toolbar :color="color" v-else-if="pasoPuntos == 'window-2'">
          <v-btn icon @click="pasoPuntos = 'window-1'">
              <v-icon left >mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
              Sentido de la votación
          </v-toolbar-title>
        </v-toolbar>
        <v-toolbar :color="color" v-else-if="pasoPuntos == 'window-3'">
          <v-btn icon @click="pasoPuntos = 'window-1'">
              <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
              Nueva Intervención
          </v-toolbar-title>
        </v-toolbar>
        <v-toolbar :color="color" v-else-if="pasoPuntos == 'window-4'">
          <v-btn icon @click="pasoPuntos = 'window-1'">
              <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
              Relación de intervenciones
          </v-toolbar-title>
        </v-toolbar>
        <v-toolbar :color="color" v-else-if="pasoPuntos == 'window-41'">
          <v-btn icon @click="pasoPuntos = 'window-1'">
              <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
              Edición de la intervención
          </v-toolbar-title>
        </v-toolbar>
        <v-toolbar :color="color" v-else-if="pasoPuntos == 'window-5'">
          <v-btn icon @click="pasoPuntos = 'window-1'">
              <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
              Nuevo incidente
          </v-toolbar-title>
        </v-toolbar>
        <v-toolbar :color="color" v-else-if="pasoPuntos == 'window-6'">
          <v-btn icon @click="pasoPuntos = 'window-1'">
              <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
            Relación de incidentes
          </v-toolbar-title>
        </v-toolbar>
        <v-toolbar :color="color" v-else-if="pasoPuntos == 'window-61'">
          <v-btn icon @click="pasoPuntos = 'window-6'">
              <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
              Edición de incidentes
          </v-toolbar-title>
        </v-toolbar>
        <v-toolbar :color="color" v-else-if="pasoPuntos == 'window-31'">
          <v-btn icon @click="pasoPuntos = 'window-4'">
              <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
              Edición de la intervención
          </v-toolbar-title>
        </v-toolbar>
        <v-card-text class="ma-0 pa-0" >
        <!-- <v-card-text style="height: 500px;" class="ma-0 pa-0"> -->
          <v-window v-model="pasoPuntos">
              <v-window-item value="window-1">
                <v-card class="justify-center align-center text-center">
                  <v-card-text>
                      <!-- <v-virtual-scroll  :items="itemsScroll" height="360" item-height="80"> -->
                        <v-container fluid>
                          <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center my-0 py-0">
                              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="my-0 py-0">
                                <v-data-table density="compact" :color="color" item-value="idPunto" :items-per-page="itemsPerPageO" multi-sort
                                item-key="idSesion" :headers="headers3" :search="buscar2" :items="desserts3" fixed-header no-data-text="No hay datos disponibles"
                                elevation="16" transition="scale-transition" origin="center center" hide-default-footer :page="pageO" class="my-0 py-0" :height="desserts3.length < 15 ? 'auto': '400' ">
                                    <!--<template v-slot:top>
                                        <v-toolbar flat>
                                            <v-toolbar-title>Expandable Table</v-toolbar-title>
                                        </v-toolbar>
                                    </template>-->
                                    <template v-slot:[`item.descripcion`]="{ item }">
                                      <template v-if="item.cuenta == 0 || item.cuenta == null">
                                        {{  item.descripcion  }}
                                      </template>
                                      <v-card v-else-if="item.cuenta > 0" :color="coloresItem(item.cuenta)" :text="item.descripcion" variant="flat"></v-card>
                                    </template>
                                    <template v-slot:[`item.votaciones`]="{ item }">
                                        <v-btn :color="color" variant="text" icon="mdi-vote" @click="abreVotaciones(item.numPunto)" v-if="(item.cuenta == 0 || item.cuenta == null)  && item.inhabilitados == null"></v-btn>
                                        <template v-else-if="item.cuenta > 0 && item.inhabilitados == null">
                                          <v-tooltip text="Votación ya realizada" location="bottom">
                                            <template v-slot:activator="{ props }">
                                              <v-btn v-bind="props" color="red" variant="text" icon="mdi-block-helper"></v-btn>
                                            </template>
                                          </v-tooltip>
                                        </template>
                                        <v-btn color="#E53935" variant="text" icon="mdi-minus-circle" v-else-if="item.inhabilitados != null"></v-btn>
                                    </template>
                                    <template v-slot:[`item.intervenciones`]="{ item }">
                                        <v-btn color="green" variant="text" icon="mdi-plus-circle-outline" @click="abreNuevoIntervencion(item.numPunto)" v-if="item.inhabilitados == null"></v-btn>
                                        <v-btn color="#E53935" variant="text" icon="mdi-minus-circle" v-else-if=" item.inhabilitados != null"></v-btn>
                                        <v-btn color="blue" variant="text" icon="mdi-playlist-edit" @click="abreEditaIntervencion(item.numPunto)" v-if="item.inhabilitados == null"></v-btn>
                                        <v-btn color="#E53935" variant="text" icon="mdi-minus-circle" v-else-if="item.inhabilitados != null"></v-btn>
                                    </template>
                                    <template v-slot:[`item.incidentes`]="{ item }">
                                        <v-btn color="green" variant="text" icon="mdi-plus-circle-outline" @click="abreNuevoIncidente(item.numPunto)" v-if="item.inhabilitados == null"></v-btn>
                                        <v-btn color="#E53935" variant="text" icon="mdi-minus-circle" v-else-if="item.inhabilitados != null"></v-btn>
                                        <v-btn color="blue" variant="text" icon="mdi-playlist-edit" @click="abreEditaIncidente(item.numPunto)" v-if="item.inhabilitados == null"></v-btn>
                                        <v-btn color="#E53935" variant="text" icon="mdi-minus-circle" v-else-if="item.inhabilitados != null"></v-btn>
                                    </template>
                                    <template v-slot:[`item.quitarlos`]="{ item }">
                                        <v-btn color="#E53935" variant="text" icon="mdi-file-document-remove" @click="abreQuitaPunto(item)" v-if="item.puntoObligatorio == '0' && item.puntoVotar == '0'  && item.inhabilitados == null"></v-btn>
                                        <v-btn color="#E53935" variant="text" icon="mdi-lock-open" @click="abreAgregaPunto(item)" v-else-if="item.puntoObligatorio == '0' && item.puntoVotar == '0'  && item.inhabilitados != null"></v-btn>
                                        <v-chip variant="elevated" class="py-5" v-else-if="item.puntoObligatorio == '1' && item.puntoVotar == '1'" :color="color"> Obligatorio / <br> Votación requerida </v-chip>
                                        <v-chip variant="elevated" class="py-5" v-else-if="item.puntoObligatorio == '1'" :color="color"> Obligatorio </v-chip>
                                        <v-chip variant="elevated" class="py-5" v-else-if="item.puntoVotar == '1'" :color="color"> Votación requerida </v-chip>
                                    </template>
                                    <template v-slot:bottom>
                                        <v-container>
                                            <v-row class="d-flex flex-md-row flex-column text-center my-0 py-0">
                                                <v-col col="2" md="2"></v-col>
                                                <v-col col="8" md="8">
                                                    <v-pagination v-model="pageO" :color="color" rounded="circle" :length="pageCountO"></v-pagination>
                                                </v-col>
                                                <!--<v-col col="2" md="2">
                                                    <v-text-field :model-value="itemsPerPageO" class="pa-2" label="Núm. por página" type="number" min="-1" max="15" hide-details @update:model-value="itemsPerPageO = parseInt($event, 10)"></v-text-field>
                                                </v-col>-->
                                            </v-row>
                                        </v-container>
                                    </template>
                                </v-data-table>
                              </v-col>
                          </v-row>
                      </v-container>
                      <v-card-actions class="justify-center align-center text-center">
                        <v-btn variant="tonal" size="large" color="red" @click="dialogOrdenDDia = false"> Cerrar </v-btn>
                      </v-card-actions>
                      <!-- </v-virtual-scroll> -->
                      <!-- <v-container fluid class="pt-0 mt-0">
                          <v-row class="d-flex flex-sm-row flex-column justify-center align-center text-center mt-0 pt-0">
                              <v-col col="4" xs="4" sm="4" md="4" lg="4">
                                  <v-btn variant="tonal" size="large" color="red" @click="dialogOrdenDDia = false"> Cerrar </v-btn>
                              </v-col>
                          </v-row>
                      </v-container> -->
                  </v-card-text>
              </v-card>
              </v-window-item>
              <v-window-item value="window-2">
                <!-- <v-toolbar :color="color">
                  <v-btn icon @click="pasoPuntos = 'window-1'">
                      <v-icon left >mdi-arrow-left</v-icon>
                  </v-btn>
                  <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
                      Sentido de la votación
                  </v-toolbar-title>
                </v-toolbar> -->
                <v-card>
                  <v-card-text>
                    <!-- <v-virtual-scroll  :items="itemsScroll" height="500" item-height="10">  -->
                    <v-container fluid>
                        <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                          <v-col col="0" xs="0" sm="0" md="1" lg="1" class="text-center"></v-col>
                          <v-col col="12" xs="12" sm="12" md="10" lg="10" class="text-center">
                            <v-form ref="formVotaciones">
                              <v-table fixed-header widht="400">
                                <thead>
                                  <tr>
                                    <th class="text-center">
                                      Siglas
                                    </th>
                                    <th class="text-center">
                                      Cargo
                                    </th>
                                    <th class="text-center">
                                      Nombre
                                    </th>
                                    <th class="text-center">
                                      Votación
                                    </th>            
                                  </tr>
                                </thead>
                                <tbody>
                                  <tr v-for="integrantes in dessertsIntegrantes" :key="integrantes['idIntegrante']">
                                    <template v-if="integrantes['siglas'] != 'SC'">
                                      <td class="text-center">{{ integrantes['siglas'] }}</td>
                                      <td class="text-center">{{ integrantes['cargo'] }}</td>
                                      <td class="text-center">{{ integrantes['nombre'] }}</td>
                                      <td class="text-center">
                                        <v-radio-group class="d-inline justify-center align-center text-center"  inline :color="color" v-model="integrantes['votacion']" >
                                          <v-radio label="A favor" value="F"></v-radio>
                                          <v-radio label="En contra" value="C"></v-radio>
                                          <v-radio label="Abstención" value="E"></v-radio>
                                        </v-radio-group>
                                      </td>
                                    </template>
                                  </tr>
                                </tbody>
                              </v-table>
                            </v-form>
                          </v-col>     
                          <v-col col="0" xs="0" sm="0" md="1" lg="1" class="text-center "></v-col>
                          <v-col col="12" xs="12" sm="12" md="2" lg="2" class="text-center ">
                            Observaciones:
                          </v-col>
                          <v-col col="12" xs="12" sm="12" md="5" lg="5" >
                            <v-textarea type="text" :color="color" v-model="formVotacion.observaciones" :counter="2000" min="1" label="Observaciones de la votación" variant="outlined" 
                            :error-messages="observacionesVotacionErrors" @change="v$.formVotacion.observaciones.$touch()" @blur="v$.formVotacion.observaciones.$touch()" ref="observacionesVota" maxlength ="2000"></v-textarea>
                          </v-col>                                
                        </v-row>
                    </v-container>
                    <!-- </v-virtual-scroll> -->
                    <!-- <v-container fluid>
                      <v-row class="d-flex flex-sm-row flex-column justify-center align-center text-center my-0 py-0">
                        <v-card-actions class="d-flex justify-center">
                          <v-btn variant="tonal" size="large" color="red" @click="dialogOrdenDDia = false"> Cerrar </v-btn>
                          <v-btn variant="tonal" size="large" :color="color" @click="validaVotacion"> Guardar </v-btn>
                        </v-card-actions>
                      </v-row>
                    </v-container> -->
                  </v-card-text>
                  <v-card-actions class="d-flex justify-center">
                    <v-btn variant="tonal" size="large" color="red" @click="pasoPuntos = 'window-1'"> Regresar </v-btn>
                    <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="validaVotacionPrevio"> Guardar </v-btn>
                  </v-card-actions> 
                </v-card>
              </v-window-item>
              <v-window-item value="window-3">
                <!-- <v-toolbar :color="color">
                  <v-btn icon @click="pasoPuntos = 'window-1'">
                      <v-icon>mdi-arrow-left</v-icon>
                  </v-btn>
                  <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
                      Nueva Intervención
                  </v-toolbar-title>
                </v-toolbar> -->
                <v-card >
                    <v-card-text class="mt-0 pt-0">
                      <!-- <v-virtual-scroll  :items="itemsScroll" height="500" item-height="10" >  -->
                          <v-container fluid  class="mt-0">
                              <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                                <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                  Consejeros involucrados
                                </v-col>
                                <v-col col="12" xs="12" sm="12" md="9" lg="9" class="text-center">
                                  <v-table fixed-header widht="400">
                                    <thead>
                                      <tr>
                                        <th class="text-center">
                                          Nombre
                                        </th>
                                        <th class="text-center">
                                          Cargo
                                        </th>
                                        <th class="text-center">
                                          Asistencia
                                        </th>                        
                                      </tr>
                                    </thead>
                                    <tbody>
                                      <tr v-for="integrantes in dessertsIntegrantes" :key="integrantes['idIntegrante']">
                                        <!-- <template v-if="integrantes['siglas'] != 'SC'"> -->
                                          <td class="text-center">{{ integrantes['nombre'] }}</td>
                                          <td class="text-center">{{ integrantes['siglas'] }}</td>
                                          <td class="text-center">
                                            <v-checkbox class="d-inline-flex" :color="color" v-model="integrantes['asistencia']"></v-checkbox>
                                          </td>
                                        <!-- </template> -->
                                      </tr>
                                    </tbody>
                                  </v-table>
                                </v-col>
                                <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                  <v-form ref="formNuevaIntervencion">
                                    <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                      Partidos Políticos involucrados
                                    </v-col>
                                    <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                                        <v-row class="d-flex text-center justify-center align-content-center ">
                                          <template v-for="integrantes in dessertsPartidos" :key="integrantes.idIntegrante">
                                            <v-col cols="auto" xs="3" sm="3" md="3" lg="3" class="d-flex text-center justify-center align-content-center ">
                                              <v-table fixed-header>
                                                <thead>
                                                  <tr>
                                                    <th class="text-center justify-center align-content-center">
                                                      {{ integrantes['siglasPartido'] }}
                                                      <v-tooltip text="Limpiar asistencia" location="top">
                                                        <template v-slot:activator="{ props }">
                                                          <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>  
                                                        </template>
                                                      </v-tooltip>
                                                    </th>                     
                                                  </tr>
                                                </thead>
                                                <tbody >
                                                  <tr>
                                                    <td class="text-center justify-center align-content-center">
                                                      <v-radio-group  inline :color="color" v-model="integrantes['asistencia']">
                                                        <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                                          <template v-slot:activator="{ props }">
                                                            <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                                          </template>
                                                        </v-tooltip> -->
                                                        <!-- <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                                          <template v-slot:activator="{ props }">
                                                            <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                                          </template>
                                                        </v-tooltip> -->
                                                        <v-radio label="P" value="P"></v-radio>
                                                        <v-radio label="S" value="S"></v-radio>
                                                      </v-radio-group>
                                                    </td>
                                                  </tr>
                                                </tbody>
                                              </v-table>
                                            </v-col>
                                          </template>
                                        </v-row>
                                    </v-col>
                                    <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                      Candidaturas Sin Partido involucradas
                                    </v-col>
                                    <v-container fluid>
                                      <v-row class="d-flex text-center justify-center align-content-center ">
                                        <template v-for="integrantes in dessertsIndependientes" :key="integrantes.idIntegrante">
                                          <v-col cols="auto" xs="3" sm="3" md="3" lg="3">
                                            <div >
                                              <v-table fixed-header >
                                                <thead>
                                                  <tr>
                                                    <th class="text-center">
                                                      {{ integrantes['nombrePartido'] }} <br>( {{ integrantes['siglasPartido'] }} )
                                                      <v-tooltip text="Limpiar asistencia" location="top">
                                                        <template v-slot:activator="{ props }">
                                                          <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>
                                                        </template>
                                                      </v-tooltip>
                                                    </th>                     
                                                  </tr>
                                                </thead>
                                                <tbody>
                                                  <tr >
                                                    <td class="text-center justify-center align-content-center">
                                                      <v-radio-group inline :color="color" v-model="integrantes['asistencia']">
                                                        <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                                          <template v-slot:activator="{ props }">
                                                            <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                                          </template>
                                                        </v-tooltip>
                                                        <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                                          <template v-slot:activator="{ props }">
                                                            <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                                          </template>
                                                        </v-tooltip> -->
                                                        <v-radio label="P" value="P" ></v-radio>
                                                        <v-radio label="S" value="S" ></v-radio>
                                                      </v-radio-group>
                                                    </td>
                                                  </tr>
                                                </tbody>
                                              </v-table>
                                            </div>
                                          </v-col>
                                        </template>
                                      </v-row>
                                    </v-container>
                                  </v-form>
                                </v-col>

                                <v-col col="12" xs="12" sm="12" md="2" lg="2"></v-col>
                                <v-col col="12" xs="12" sm="12" md="2" lg="2">
                                    <v-btn size="small" variant="text" icon="mdi-minus" :color="color" @click="decrement"></v-btn>
                                </v-col>
                                <v-col col="12" xs="12" sm="12" md="3" lg="3">
                                    <v-col class="text-left">
                                        <span
                                            class="text-h2 font-weight-light"
                                            v-text="numIntervencion"
                                        ></span>
                                        <span class="subheading font-weight-light me-1"> Número de intervención</span>
                                    </v-col>
                                </v-col>
                                <v-col col="12" xs="12" sm="12" md="2" lg="2">
                                    <v-btn size="small" variant="text" icon="mdi-plus" :color="color" @click="increment"></v-btn>
                                </v-col>
                                <v-col col="12" xs="12" sm="12" md="2" lg="2"></v-col>
                                <v-col col="12" xs="12" sm="12" md="5" lg="5" class="text-center">
                                  Descripción del evento:
                                </v-col>
                                <v-col col="12" xs="12" sm="12" md="7" lg="7">
                                  <v-textarea type="text" :color="color" v-model="form2.descripcion" :counter="2000" min="1" label="Indicar una descripción del evento o en su caso, la leyenda sin descripción" variant="outlined" 
                                  :error-messages="descripcionErrors" @change="v$.form2.descripcion.$touch()" @blur="v$.form2.descripcion.$touch()" ref="descripcion1" maxlength ="2000"></v-textarea>
                                </v-col>   
                                <v-col col="12" xs="12" sm="12" md="5" lg="5" class="text-center">
                                  Réplica:
                                </v-col>
                                <v-col col="12" xs="12" sm="12" md="7" lg="7">
                                  <v-textarea type="text" :color="color" v-model="form2.replica" :counter="2000" min="1" label="Indicar la réplica del evento o en su caso, la leyenda sin réplica" variant="outlined" 
                                  :error-messages="replicaErrors" @change="v$.form2.replica.$touch()" @blur="v$.form2.replica.$touch()" ref="replica1" maxlength ="2000"></v-textarea>
                                </v-col>                                 
                              </v-row>
                          </v-container>
                      <!-- </v-virtual-scroll> -->
                    </v-card-text>
                    <v-card-actions class="justify-center align-center text-center">
                      <v-btn variant="tonal" size="large" color="red" @click="pasoPuntos = 'window-1'"> Regresar </v-btn>
                      <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="validaIntervencion"> Guardar </v-btn>
                    </v-card-actions>
                </v-card>
              </v-window-item>
              <v-window-item value="window-4">
                <!-- <v-toolbar :color="color">
                  <v-btn icon @click="pasoPuntos = 'window-1'">
                      <v-icon>mdi-arrow-left</v-icon>
                  </v-btn>
                  <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
                      Relación de intervenciones
                  </v-toolbar-title>
                </v-toolbar> -->
                <v-card>
                  <v-card-text>
                    <!-- <v-virtual-scroll  :items="itemsScroll" height="375" item-height="10">  -->
                    <v-container fluid>
                        <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                          <v-col col="12" xs="12" sm="12" md="12" lg="12" class="my-0 py-0">
                            <v-data-table density="compact" :color="color" :items-per-page="itemsPerPageInt" multi-sort no-data-text="No hay datos disponibles"
                            item-key="idIntervencion" :headers="headersIntervencion" :items="dessertsIntervencion" height="240" fixed-header
                            elevation="16" transition="scale-transition" origin="center center" hide-default-footer :page="pageInt" class="my-0 py-0">
                                <!--<template v-slot:top>
                                    <v-toolbar flat>
                                        <v-toolbar-title>Expandable Table</v-toolbar-title>
                                    </v-toolbar>
                                </template>-->
                                <template v-slot:[`item.editar`]="{ item }">
                                  <v-btn :color="color" variant="text" icon="mdi-pencil" @click="consultaIntervencion(item)"></v-btn>
                                </template>
                                <template v-slot:[`item.quitar`]="{ item }">
                                    <v-btn color="red" variant="text" icon="mdi-delete-empty" @click="eliminaIntervencion(item.idIntervencion)"></v-btn>
                                </template>
                                <template v-slot:bottom>
                                    <v-container>
                                        <v-row class="d-flex flex-md-row flex-column text-center my-0 py-0">
                                            <v-col col="2" md="2"></v-col>
                                            <v-col col="8" md="8">
                                                <v-pagination v-model="pageInt" :color="color" rounded="circle" :length="pageCountInt"></v-pagination>
                                            </v-col>
                                            <!--<v-col col="2" md="2">
                                                <v-text-field :model-value="itemsPerPageO" class="pa-2" label="Núm. por página" type="number" min="-1" max="15" hide-details @update:model-value="itemsPerPageO = parseInt($event, 10)"></v-text-field>
                                            </v-col>-->
                                        </v-row>
                                    </v-container>
                                </template>
                            </v-data-table>
                          </v-col>                          
                        </v-row>
                    </v-container>
                    <!-- </v-virtual-scroll> -->
                  </v-card-text>
                  <v-card-actions class="d-flex justify-center">
                    <v-btn variant="tonal" size="large" color="red" @click="pasoPuntos = 'window-1'"> Regresar </v-btn>
                  </v-card-actions> 
                </v-card>
              </v-window-item>
              <v-window-item value="window-41">
                <!-- <v-toolbar :color="color">
                  <v-btn icon @click="pasoPuntos = 'window-1'">
                      <v-icon>mdi-arrow-left</v-icon>
                  </v-btn>
                  <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
                      Edición de la intervención
                  </v-toolbar-title>
                </v-toolbar> -->
                <v-card>
                    <v-card-text>
                        <!-- <v-virtual-scroll  :items="itemsScroll" height="375" item-height="10">  -->
                            <v-container fluid>
                                <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                                                                                      
                                </v-row>
                            </v-container>
                        <!-- </v-virtual-scroll> -->
                        <v-container fluid>
                            <v-row class="d-flex flex-sm-row flex-column justify-center align-center text-center my-0 py-0">
                                
                            </v-row>
                        </v-container>
                    </v-card-text>
                </v-card>
              </v-window-item>
              <v-window-item value="window-5">
                <!-- <v-toolbar :color="color">
                  <v-btn icon @click="pasoPuntos = 'window-1'">
                      <v-icon>mdi-arrow-left</v-icon>
                  </v-btn>
                  <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
                      Nuevo incidente
                  </v-toolbar-title>
                </v-toolbar> -->
                <v-card>
                  <v-card-text>
                    <!-- <v-virtual-scroll  :items="itemsScroll" height="500" item-height="10">  -->
                      <v-container fluid  >
                          <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                            <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                              Consejeros involucrados
                            </v-col>
                            <v-col col="12" xs="12" sm="12" md="9" lg="9" class="text-center">
                              <v-table fixed-header widht="400">
                                <thead>
                                  <tr>
                                    <th class="text-center">
                                      Nombre
                                    </th>
                                    <th class="text-center">
                                      Cargo
                                    </th>
                                    <th class="text-center">
                                      Asistencia
                                    </th>                        
                                  </tr>
                                </thead>
                                <tbody>
                                  <tr v-for="integrantes in dessertsIntegrantes" :key="integrantes['idIntegrante']">
                                    <td class="text-center">{{ integrantes['nombre'] }}</td>
                                    <td class="text-center">{{ integrantes['siglas'] }}</td>
                                    <td class="text-center">
                                      <v-checkbox class="d-inline-flex" :color="color" v-model="integrantes['asistencia']"></v-checkbox>
                                    </td>
                                  </tr>
                                </tbody>
                              </v-table>
                            </v-col>
                            <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                              <v-form ref="formNuevoIncidente">
                                <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                  Partidos Políticos involucrados
                                </v-col>
                                <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                                  <v-container fluid>
                                    <v-row class="d-flex text-center justify-center align-content-center ">
                                      <template v-for="integrantes in dessertsPartidos" :key="integrantes.idIntegrante">
                                        <v-col cols="auto" xs="3" sm="3" md="3" lg="3" class="d-flex text-center justify-center align-content-center ">
                                          <v-table fixed-header>
                                            <thead>
                                              <tr>
                                                <th class="text-center justify-center align-content-center">
                                                  {{ integrantes['siglasPartido'] }}
                                                  <v-tooltip text="Limpiar asistencia" location="top">
                                                    <template v-slot:activator="{ props }">
                                                      <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>  
                                                    </template>
                                                  </v-tooltip>
                                                </th>                     
                                              </tr>
                                            </thead>
                                            <tbody >
                                              <tr>
                                                <td class="text-center justify-center align-content-center">
                                                  <v-radio-group  inline :color="color" v-model="integrantes['asistencia']">
                                                    <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                                      <template v-slot:activator="{ props }">
                                                        <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                                      </template>
                                                    </v-tooltip> -->
                                                    <!-- <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                                      <template v-slot:activator="{ props }">
                                                        <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                                      </template>
                                                    </v-tooltip> -->
                                                    <v-radio label="P" value="P"></v-radio>
                                                    <v-radio label="S" value="S"></v-radio>
                                                  </v-radio-group>
                                                </td>
                                              </tr>
                                            </tbody>
                                          </v-table>
                                        </v-col>
                                      </template>
                                    </v-row>
                                  </v-container>
                                </v-col>
                                <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                  Candidaturas Sin Partido involucradas
                                </v-col>
                                <v-container fluid>
                                  <v-row class="d-flex text-center justify-center align-content-center ">
                                    <template v-for="integrantes in dessertsIndependientes" :key="integrantes.idIntegrante">
                                      <v-col cols="auto" xs="3" sm="3" md="3" lg="3">
                                        <div >
                                          <v-table fixed-header >
                                            <thead>
                                              <tr>
                                                <th class="text-center">
                                                  {{ integrantes['nombrePartido'] }} <br>( {{ integrantes['siglasPartido'] }} )
                                                  <v-tooltip text="Limpiar asistencia" location="top">
                                                    <template v-slot:activator="{ props }">
                                                      <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>
                                                    </template>
                                                  </v-tooltip>
                                                </th>                     
                                              </tr>
                                            </thead>
                                            <tbody>
                                              <tr >
                                                <td class="text-center justify-center align-content-center">
                                                  <v-radio-group inline :color="color" v-model="integrantes['asistencia']">
                                                    <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                                      <template v-slot:activator="{ props }">
                                                        <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                                      </template>
                                                    </v-tooltip>
                                                    <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                                      <template v-slot:activator="{ props }">
                                                        <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                                      </template>
                                                    </v-tooltip> -->
                                                    <v-radio label="P" value="P" ></v-radio>
                                                    <v-radio label="S" value="S" ></v-radio>
                                                  </v-radio-group>
                                                </td>
                                              </tr>
                                            </tbody>
                                          </v-table>
                                        </div>
                                      </v-col>
                                    </template>
                                  </v-row>
                                </v-container>
                              </v-form>
                            </v-col>

                            <v-col col="12" xs="12" sm="12" md="2" lg="2"></v-col>
                            <v-col col="12" xs="12" sm="12" md="2" lg="2">
                                <v-btn size="small" variant="text" icon="mdi-minus" :color="color" @click="decrement3"></v-btn>
                            </v-col>
                            <v-col col="12" xs="12" sm="12" md="3" lg="3">
                                <v-col class="text-left">
                                    <span
                                        class="text-h2 font-weight-light"
                                        v-text="numIncidente"
                                    ></span>
                                    <span class="subheading font-weight-light me-1"> Número de incidente</span>
                                </v-col>
                            </v-col>
                            <v-col col="12" xs="12" sm="12" md="2" lg="2">
                                <v-btn size="small" variant="text" icon="mdi-plus" :color="color" @click="increment3"></v-btn>
                            </v-col>
                            <v-col col="12" xs="12" sm="12" md="2" lg="2"></v-col>
                            <v-col col="12" xs="12" sm="12" md="5" lg="5" class="text-center">
                              Descripción del evento:
                            </v-col>
                            <v-col col="12" xs="12" sm="12" md="7" lg="7">
                              <v-textarea type="text" :color="color" v-model="form31.descripcion" :counter="2000" min="1" label="Indicar una descripción del evento o en su caso, la leyenda sin descripción" variant="outlined" 
                              :error-messages="descripcion3Errors" @change="v$.form31.descripcion.$touch()" @blur="v$.form31.descripcion.$touch()" ref="descripcion1" maxlength ="2000"></v-textarea>
                            </v-col>   
                            <v-col col="12" xs="12" sm="12" md="5" lg="5" class="text-center">
                              Réplica:
                            </v-col>
                            <v-col col="12" xs="12" sm="12" md="7" lg="7">
                              <v-textarea type="text" :color="color" v-model="form31.replica" :counter="2000" min="1" label="Indicar la réplica del evento o en su caso, la leyenda sin réplica" variant="outlined" 
                              :error-messages="replica3Errors" @change="v$.form31.replica.$touch()" @blur="v$.form31.replica.$touch()" ref="replica1" maxlength ="2000"></v-textarea>
                            </v-col>                                 
                          </v-row>
                      </v-container>
                    <!-- </v-virtual-scroll> -->
                  </v-card-text>
                  <v-card-actions class="justify-center align-center text-center">
                    <v-btn variant="tonal" size="large" color="red" @click="pasoPuntos = 'window-1'"> Regresar </v-btn>
                    <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="validaIncidente"> Guardar </v-btn>
                  </v-card-actions>
                </v-card>
              </v-window-item>
              <v-window-item value="window-6">
                <!-- <v-toolbar :color="color">
                  <v-btn icon @click="pasoPuntos = 'window-1'">
                      <v-icon>mdi-arrow-left</v-icon>
                  </v-btn>
                  <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
                    Relación de incidentes
                  </v-toolbar-title>
                </v-toolbar> -->
                <v-card>
                  <v-card-text>
                    <!-- <v-virtual-scroll  :items="itemsScroll" height="375" item-height="10">  -->
                    <v-container fluid>
                        <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                          <v-col col="12" xs="12" sm="12" md="12" lg="12" class="my-0 py-0">
                            <v-data-table density="compact" :color="color" item-value="idIncidente" :items-per-page="itemsPerPageInc" multi-sort
                            item-key="idIncidente" :headers="headersIncidente" :items="dessertsIncidente" height="240" fixed-header no-data-text="No hay datos disponibles"
                            elevation="16" transition="scale-transition" origin="center center" hide-default-footer :page="pageInc" class="my-0 py-0">
                                <!--<template v-slot:top>
                                    <v-toolbar flat>
                                        <v-toolbar-title>Expandable Table</v-toolbar-title>
                                    </v-toolbar>
                                </template>-->
                                <template v-slot:[`item.editar`]="{ item }">
                                  <v-btn :color="color" variant="text" icon="mdi-pencil" @click="consultaIncidente(item)"></v-btn>
                                </template>
                                <template v-slot:[`item.quitar`]="{ item }">
                                    <v-btn color="red" variant="text" icon="mdi-delete-empty" @click="eliminaIncidente(item.idIncidente)"></v-btn>
                                </template>
                                <template v-slot:bottom>
                                    <v-container>
                                        <v-row class="d-flex flex-md-row flex-column text-center my-0 py-0">
                                            <v-col col="2" md="2"></v-col>
                                            <v-col col="8" md="8">
                                                <v-pagination v-model="pageInc" :color="color" rounded="circle" :length="pageCountInc"></v-pagination>
                                            </v-col>
                                            <!--<v-col col="2" md="2">
                                                <v-text-field :model-value="itemsPerPageO" class="pa-2" label="Núm. por página" type="number" min="-1" max="15" hide-details @update:model-value="itemsPerPageO = parseInt($event, 10)"></v-text-field>
                                            </v-col>-->
                                        </v-row>
                                    </v-container>
                                </template>
                            </v-data-table>
                          </v-col>                                             
                        </v-row>
                    </v-container>
                    <!-- </v-virtual-scroll> -->
                  </v-card-text>
                  <v-card-actions class="d-flex justify-center">
                    <v-btn variant="tonal" size="large" color="red" @click="pasoPuntos = 'window-1'"> Regresar </v-btn>
                  </v-card-actions>
                </v-card>
              </v-window-item>
              <v-window-item value="window-61">
                <!-- <v-toolbar :color="color">
                  <v-btn icon @click="pasoPuntos = 'window-1'">
                      <v-icon>mdi-arrow-left</v-icon>
                  </v-btn>
                  <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
                      Edición de incidentes
                  </v-toolbar-title>
                </v-toolbar> -->
                <v-card>
                  <v-card-text>
                      <!-- <v-virtual-scroll  :items="itemsScroll" height="375" item-height="10">  -->
                          <v-container fluid>
                            <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                Consejeros involucrados
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="9" lg="9" class="text-center">
                                <v-table fixed-header widht="400">
                                  <thead>
                                    <tr>
                                      <th class="text-center">
                                        Nombre
                                      </th>
                                      <th class="text-center">
                                        Cargo
                                      </th>
                                      <th class="text-center">
                                        Asistencia
                                      </th>                        
                                    </tr>
                                  </thead>
                                  <tbody>
                                    <tr v-for="integrantes in dessertsIntegrantes" :key="integrantes['idIntegrante']">
                                      <td class="text-center">{{ integrantes['nombre'] }}</td>
                                      <td class="text-center">{{ integrantes['siglas'] }}</td>
                                      <td class="text-center">
                                        <v-checkbox class="d-inline-flex" :color="color" v-model="integrantes['asistencia']"></v-checkbox>
                                      </td>
                                    </tr>
                                  </tbody>
                                </v-table>
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                <v-form ref="formNuevoIncidente">
                                  <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                    Partidos Políticos involucrados
                                  </v-col>
                                  <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                                    <v-container fluid>
                                      <v-row class="d-flex text-center justify-center align-content-center ">
                                        <template v-for="integrantes in dessertsPartidos" :key="integrantes.idIntegrante">
                                          <v-col cols="auto" xs="3" sm="3" md="3" lg="3" class="d-flex text-center justify-center align-content-center ">
                                            <v-table fixed-header>
                                              <thead>
                                                <tr>
                                                  <th class="text-center justify-center align-content-center">
                                                    {{ integrantes['siglasPartido'] }}
                                                    <v-tooltip text="Limpiar asistencia" location="top">
                                                      <template v-slot:activator="{ props }">
                                                        <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>  
                                                      </template>
                                                    </v-tooltip>
                                                  </th>                     
                                                </tr>
                                              </thead>
                                              <tbody >
                                                <tr>
                                                  <td class="text-center justify-center align-content-center">
                                                    <v-radio-group  inline :color="color" v-model="integrantes['asistencia']">
                                                      <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                                        <template v-slot:activator="{ props }">
                                                          <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                                        </template>
                                                      </v-tooltip> -->
                                                      <!-- <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                                        <template v-slot:activator="{ props }">
                                                          <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                                        </template>
                                                      </v-tooltip> -->
                                                      <v-radio label="P" value="P"></v-radio>
                                                      <v-radio label="S" value="S"></v-radio>
                                                    </v-radio-group>
                                                  </td>
                                                </tr>
                                              </tbody>
                                            </v-table>
                                          </v-col>
                                        </template>
                                      </v-row>
                                    </v-container>
                                  </v-col>
                                  <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                    Candidaturas Sin Partido involucradas
                                  </v-col>
                                  <v-container fluid>
                                    <v-row class="d-flex text-center justify-center align-content-center ">
                                      <template v-for="integrantes in dessertsIndependientes" :key="integrantes.idIntegrante">
                                        <v-col cols="auto" xs="3" sm="3" md="3" lg="3">
                                          <div >
                                            <v-table fixed-header >
                                              <thead>
                                                <tr>
                                                  <th class="text-center">
                                                    {{ integrantes['nombrePartido'] }} <br>( {{ integrantes['siglasPartido'] }} )
                                                    <v-tooltip text="Limpiar asistencia" location="top">
                                                      <template v-slot:activator="{ props }">
                                                        <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>
                                                      </template>
                                                    </v-tooltip>
                                                  </th>                     
                                                </tr>
                                              </thead>
                                              <tbody>
                                                <tr >
                                                  <td class="text-center justify-center align-content-center">
                                                    <v-radio-group inline :color="color" v-model="integrantes['asistencia']">
                                                      <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                                        <template v-slot:activator="{ props }">
                                                          <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                                        </template>
                                                      </v-tooltip>
                                                      <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                                        <template v-slot:activator="{ props }">
                                                          <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                                        </template>
                                                      </v-tooltip> -->
                                                      <v-radio label="P" value="P" ></v-radio>
                                                      <v-radio label="S" value="S" ></v-radio>
                                                    </v-radio-group>
                                                  </td>
                                                </tr>
                                              </tbody>
                                            </v-table>
                                          </div>
                                        </v-col>
                                      </template>
                                    </v-row>
                                  </v-container>
                                </v-form>
                              </v-col>

                              <v-col col="12" xs="12" sm="12" md="2" lg="2"></v-col>
                              <v-col col="12" xs="12" sm="12" md="2" lg="2">
                                  <!-- <v-btn size="small" variant="text" icon="mdi-minus" :color="color" @click="decrement3"></v-btn> -->
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="3" lg="3">
                                  <v-col class="text-left">
                                      <span
                                          class="text-h2 font-weight-light"
                                          v-text="numIncidente"
                                      ></span>
                                      <span class="subheading font-weight-light me-1"> Número de incidente</span>
                                  </v-col>
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="2" lg="2">
                                  <!-- <v-btn size="small" variant="text" icon="mdi-plus" :color="color" @click="increment3"></v-btn> -->
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="2" lg="2"></v-col>
                              <v-col col="12" xs="12" sm="12" md="5" lg="5" class="text-center">
                                Descripción del evento:
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                                <v-textarea type="text" :color="color" v-model="form31.descripcion" :counter="2000" min="1" label="Indicar una descripción del evento o en su caso, la leyenda sin descripción" variant="outlined" 
                                :error-messages="descripcion3Errors" @change="v$.form31.descripcion.$touch()" @blur="v$.form31.descripcion.$touch()" ref="descripcion1" maxlength ="2000"></v-textarea>
                              </v-col>   
                              <v-col col="12" xs="12" sm="12" md="5" lg="5" class="text-center">
                                Réplica:
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                                <v-textarea type="text" :color="color" v-model="form31.replica" :counter="2000" min="1" label="Indicar la réplica del evento o en su caso, la leyenda sin réplica" variant="outlined" 
                                :error-messages="replica3Errors" @change="v$.form31.replica.$touch()" @blur="v$.form31.replica.$touch()" ref="replica1" maxlength ="2000"></v-textarea>
                              </v-col>                                 
                            </v-row>
                          </v-container>
                      <!-- </v-virtual-scroll> -->
                  </v-card-text>
                  <v-card-actions class="justify-center align-center text-center">
                    <v-btn variant="tonal" size="large" color="red" @click="pasoPuntos = 'window-1'"> Regresar </v-btn>
                    <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="validaIncidenteEdicion"> Guardar </v-btn>
                  </v-card-actions>
                </v-card>
              </v-window-item>
              <v-window-item value="window-31">
                <!-- <v-toolbar :color="color">
                  <v-btn icon @click="pasoPuntos = 'window-4'">
                      <v-icon>mdi-arrow-left</v-icon>
                  </v-btn>
                  <v-toolbar-title class="mx-auto d-flex  text-center text-wrap justify-center align-center">
                      Edición de la intervención
                  </v-toolbar-title>
                </v-toolbar> -->
                <v-card >
                  <v-card-text class="mt-0 pt-0">
                    <!-- <v-virtual-scroll  :items="itemsScroll" height="500" item-height="10" >  -->
                        <v-container fluid  class="mt-0">
                            <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                Consejeros involucrados
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="9" lg="9" class="text-center">
                                <v-table fixed-header widht="400">
                                  <thead>
                                    <tr>
                                      <th class="text-center">
                                        Nombre
                                      </th>
                                      <th class="text-center">
                                        Cargo
                                      </th>
                                      <th class="text-center">
                                        Asistencia
                                      </th>                        
                                    </tr>
                                  </thead>
                                  <tbody>
                                    <tr v-for="integrantes in dessertsIntegrantes" :key="integrantes['idIntegrante']">
                                      <td class="text-center">{{ integrantes['nombre'] }}</td>
                                      <td class="text-center">{{ integrantes['siglas'] }}</td>
                                      <td class="text-center">
                                        <v-checkbox class="d-inline-flex" :color="color" v-model="integrantes['asistencia']"></v-checkbox>
                                      </td>
                                    </tr>
                                  </tbody>
                                </v-table>
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                <v-form ref="formEditaIntervencion">
                                  <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                    Partidos Políticos involucrados
                                  </v-col>
                                  <v-container fluid>
                                    <v-row class="d-flex text-center justify-center align-content-center ">
                                      <template v-for="integrantes in dessertsPartidos" :key="integrantes.idIntegrante">
                                        <v-col cols="auto" xs="3" sm="3" md="3" lg="3" class="d-flex text-center justify-center align-content-center ">
                                          <v-table fixed-header>
                                            <thead>
                                              <tr>
                                                <th class="text-center justify-center align-content-center">
                                                  {{ integrantes['siglasPartido'] }}
                                                  <v-tooltip text="Limpiar asistencia" location="top">
                                                    <template v-slot:activator="{ props }">
                                                      <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>  
                                                    </template>
                                                  </v-tooltip>
                                                </th>                     
                                              </tr>
                                            </thead>
                                            <tbody >
                                              <tr>
                                                <td class="text-center justify-center align-content-center">
                                                  <v-radio-group  inline :color="color" v-model="integrantes['asistencia']">
                                                    <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                                      <template v-slot:activator="{ props }">
                                                        <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                                      </template>
                                                    </v-tooltip> -->
                                                    <!-- <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                                      <template v-slot:activator="{ props }">
                                                        <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                                      </template>
                                                    </v-tooltip> -->
                                                    <v-radio label="P" value="P"></v-radio>
                                                    <v-radio label="S" value="S"></v-radio>
                                                  </v-radio-group>
                                                </td>
                                              </tr>
                                            </tbody>
                                          </v-table>
                                        </v-col>
                                      </template>
                                    </v-row>
                                  </v-container>
                                  <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center"> 
                                    Candidaturas Sin Partido involucradas
                                  </v-col>
                                  <v-container fluid>
                                    <v-row class="d-flex text-center justify-center align-content-center ">
                                      <template v-for="integrantes in dessertsIndependientes" :key="integrantes.idIntegrante">
                                        <v-col cols="auto" xs="3" sm="3" md="3" lg="3">
                                          <div >
                                            <v-table fixed-header >
                                              <thead>
                                                <tr>
                                                  <th class="text-center">
                                                    {{ integrantes['nombrePartido'] }} <br>( {{ integrantes['siglasPartido'] }} )
                                                    <v-tooltip text="Limpiar asistencia" location="top">
                                                      <template v-slot:activator="{ props }">
                                                        <v-btn color="#E53935" v-bind="props" size="small" variant="text" icon="mdi-eraser" @click="integrantes['asistencia'] = ''"></v-btn>
                                                      </template>
                                                    </v-tooltip>
                                                  </th>                     
                                                </tr>
                                              </thead>
                                              <tbody>
                                                <tr >
                                                  <td class="text-center justify-center align-content-center">
                                                    <v-radio-group inline :color="color" v-model="integrantes['asistencia']" >
                                                      <!-- <v-tooltip :text="integrantes.nombre1 + ' ' + integrantes.primerApellido1 + ' ' + integrantes.segundoApellido1"  location="bottom">
                                                        <template v-slot:activator="{ props }">
                                                          <v-radio v-bind="props"  label="P" value="P" :rules="rulesObligatory"></v-radio>
                                                        </template>
                                                      </v-tooltip>
                                                      <v-tooltip :text="integrantes.nombre2 + ' ' + integrantes.primerApellido2 + ' ' + integrantes.segundoApellido2"  location="bottom">
                                                        <template v-slot:activator="{ props }">
                                                          <v-radio v-bind="props" label="S" value="S" :rules="rulesObligatory"></v-radio>
                                                        </template>
                                                      </v-tooltip> -->
                                                      <v-radio label="P" value="P" ></v-radio>
                                                      <v-radio label="S" value="S" ></v-radio>
                                                    </v-radio-group>
                                                  </td>
                                                </tr>
                                              </tbody>
                                            </v-table>
                                          </div>
                                        </v-col>
                                      </template>
                                    </v-row>
                                  </v-container>
                                </v-form>
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="2" lg="2"></v-col>
                              <v-col col="12" xs="12" sm="12" md="2" lg="2">
                                  <v-btn size="small" variant="text" icon="mdi-minus" :color="color" @click="decrement"></v-btn>
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="3" lg="3">
                                  <v-col class="text-left">
                                      <span
                                          class="text-h2 font-weight-light"
                                          v-text="numIntervencion"
                                      ></span>
                                      <span class="subheading font-weight-light me-1"> Número de intervención</span>
                                  </v-col>
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="2" lg="2">
                                  <v-btn size="small" variant="text" icon="mdi-plus" :color="color" @click="increment"></v-btn>
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="2" lg="2"></v-col>
                              <v-col col="12" xs="12" sm="12" md="5" lg="5" class="text-center">
                                Descripción del evento:
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                                <v-textarea type="text" :color="color" v-model="form2.descripcion" :counter="2000" min="1" label="Indicar una descripción del evento o en su caso, la leyenda sin descripción" variant="outlined" 
                                :error-messages="descripcionErrors" @change="v$.form2.descripcion.$touch()" @blur="v$.form2.descripcion.$touch()" ref="descripcion1" maxlength ="2000"></v-textarea>
                              </v-col>   
                              <v-col col="12" xs="12" sm="12" md="5" lg="5" class="text-center">
                                Réplica:
                              </v-col>
                              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                                <v-textarea type="text" :color="color" v-model="form2.replica" :counter="2000" min="1" label="Indicar la réplica del evento o en su caso, la leyenda sin réplica" variant="outlined" 
                                :error-messages="replicaErrors" @change="v$.form2.replica.$touch()" @blur="v$.form2.replica.$touch()" ref="replica1" maxlength ="2000"></v-textarea>
                              </v-col>                                 
                            </v-row>
                        </v-container>
                    <!-- </v-virtual-scroll> -->
                  </v-card-text>
                  <v-card-actions class="justify-center align-center text-center">
                    <v-btn variant="tonal" size="large" color="red" @click="pasoPuntos = 'window-4'"> Regresar </v-btn>
                    <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="validaIntervencionEdicion"> Guardar </v-btn>
                  </v-card-actions>
                </v-card>
              </v-window-item>
          </v-window>
        </v-card-text>
        <!-- </v-card-text> -->
      </v-card>
      <v-dialog v-model="dialogQuitaPunto" persistent max-width="600" :retain-focus="false">
        <v-card class="rounded-xl">
          <v-img height="200" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
              <v-card title="Estas por inhabilitar el punto para esta sesión" class="text-white" color="rgba(0, 0, 0, 0)" variant="flat">
                  ¿Continuar?<br>
                  <v-avatar :color="color" size="50">
                      <v-icon large>
                        mdi-forum-remove
                      </v-icon>
                  </v-avatar>
              </v-card>
          </v-img>
          <v-card-actions class="d-flex justify-center">
              <v-btn variant="tonal" size="large" color="red" @click="dialogQuitaPunto = false"> Cancelar </v-btn>
              <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="quitaPuntoSesion"> Inhabilitar </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialogAgregaQPunto" persistent max-width="600" :retain-focus="false">
        <v-card class="rounded-xl">
          <v-img height="200" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
              <v-card title="Estas por volver habilitar el punto para esta sesión" class="text-white" color="rgba(0, 0, 0, 0)" variant="flat">
                  ¿Continuar?<br>
                  <v-avatar :color="color" size="50">
                      <v-icon large>
                        mdi-forum-remove
                      </v-icon>
                  </v-avatar>
              </v-card>
          </v-img>
          <v-card-actions class="d-flex justify-center">
            <v-btn variant="tonal" size="large" color="red" @click="dialogAgregaQPunto = false"> Cancelar </v-btn>
            <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="habilitaPuntoSesion"> Habilitar </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialogQuitaIntervencion" persistent max-width="600" :retain-focus="false">
        <v-card class="rounded-xl">
          <v-img height="200" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
              <v-card title="Eliminar la intervención de la sesión" class="text-white text-center text-wrap" color="rgba(0, 0, 0, 0)" variant="flat">
                  ¿Continuar?<br>
                  <v-avatar :color="color" size="50">
                      <v-icon large>
                        mdi-forum-remove
                      </v-icon>
                  </v-avatar>
              </v-card>
          </v-img>
          <v-card-actions class="d-flex justify-center">
              <v-btn variant="tonal" size="large" color="red" @click="dialogQuitaIntervencion = false"> Cancelar </v-btn>
              <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="borrarIntervencion"> Eliminar </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialogAvisoVota" persistent max-width="600" :retain-focus="false">
        <v-card class="rounded-xl">
          <v-img height="200" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
            <v-card title="Atención" class="text-white text-center text-wrap" color="rgba(0, 0, 0, 0)" variant="flat">
              <p>Se tienen {{contadorVotacionIntegrantes}} votaciones del Consejo Distrital de {{contadorAsistenciaIntegrantes}} que asistieron</p>
              <p>¿Continuar con la votación?</p>
              <v-avatar :color="color" size="50">
                  <v-icon large>
                    mdi-alert
                  </v-icon>
              </v-avatar>
            </v-card>
          </v-img>
          <v-card-actions class="d-flex justify-center">
              <v-btn variant="tonal" size="large" color="red" @click="dialogAvisoVota = false"> Cancelar </v-btn>
              <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="validaVotacion"> Aceptar </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialogQuitaIncidente" persistent max-width="600" :retain-focus="false">
        <v-card class="rounded-xl">
          <v-img height="200" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
              <v-card title="Eliminar el incidente de la sesión" class="text-white" color="rgba(0, 0, 0, 0)" variant="flat">
                  ¿Continuar?<br>
                  <v-avatar :color="color" size="50">
                      <v-icon large>
                        mdi-forum-remove
                      </v-icon>
                  </v-avatar>
              </v-card>
          </v-img>
          <v-card-actions class="d-flex justify-center">
              <v-btn variant="tonal" size="large" color="red" @click="dialogQuitaIncidente = false"> Cancelar </v-btn>
              <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color" @click="borrarIncidente"> Eliminar </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-dialog>
    <!-- termina de orden de seguimiento -->
    <!--  Inicia Estado de la sesión   -->
    <v-dialog v-model="dialogEstado_sesion" scrollable max-width="80%" content-class="rounded-xl elevation-12" :retain-focus="false">
      <v-card class="rounded-xl">
        <v-img height="90" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
          <v-card title="Estado de la sesión" color="rgba(0, 0, 0, 0)" class="text-white" variant="flat"></v-card>
        </v-img>
        <v-toolbar :color="color">
          <v-btn icon @click="dialogEstado_sesion = false">
            <v-icon left >mdi-arrow-left</v-icon>
          </v-btn>
          <v-toolbar-title class="text-center">
            Verificar bien que el estado de la sesión sea el correcto
          </v-toolbar-title>
        </v-toolbar>
        <!-- <v-virtual-scroll  :items="itemsScroll" height="500" item-height="80"> -->
        <v-card-text class="justify-center align-center text-center">
          <v-container fluid class="my-0 py-0">
            <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
              <v-col col="12" xs="12" sm="12" md="5" lg="5" class="text-center">
                Selecciona el estado de la sesion:
              </v-col>
              <v-col col="12" xs="12" sm="12" md="6" lg="6">
                <v-select
                  clearable
                  label="Seleccionar una opción"
                  v-model = "form3.Estado_sesion"
                  :items="estadosSesiones"
                  item-title="name" item-value="value"
                  :color="color"
                  variant="outlined"
                  :error-messages="Estado_sesionErrors" @change="v$.form3.Estado_sesion.$touch()" @blur="v$.form3.Estado_sesion.$touch()"
                  ref="Estado_sesion"
                ></v-select>
              </v-col>
              <v-col col="12" xs="12" md="5" lg="5">Hora inicio:</v-col>
              <v-col col="12" xs="12" sm="12" md="6" lg="6">
                <v-text-field :color="color" v-model="form3.horaini_estses" label="Indicar hora inicio" :maxlength="5" 
                    variant="outlined" :error-messages="horaini_estsesErrors" ref="horaini_estses">
                  <template #prepend>
                    <v-dialog ref="dialogRegH" v-model="modal3"  persistent width="350px" overlay-opacity=".1">
                      <template v-slot:activator="{ props }">
                        <v-btn  size="x-large" v-bind="props" density="compact" :color="color"><v-icon>mdi-clock-time-eight-outline</v-icon></v-btn>
                      </template>
                      <v-card class="align-center mx-8">
                        <time-picker :color="color" v-model="form3.horaini_estses" automatic></time-picker>
                        <v-card-actions>
                          <v-btn variant="text" :color="color" @click="modal3 = false">Aceptar</v-btn>
                        </v-card-actions>
                      </v-card>
                    </v-dialog>
                  </template>  
                </v-text-field>
                <!-- <v-dialog ref="dialogRegH" v-model="modal3"  persistent width="350px" overlay-opacity=".1">
                  <template v-slot:activator="{ props }">
                    <v-text-field readonly :color="color" v-model="form3.horaini_estses" prepend-icon="mdi-clock-time-eight-outline" label="Indicar hora inicio" 
                    variant="outlined" v-bind="props" :error-messages="horaini_estsesErrors" ref="horaini_estses"></v-text-field>
                  </template>
                  <v-card class="align-center mx-8">
                    <time-picker :color="color" v-model="form3.horaini_estses" automatic></time-picker>
                    <v-card-actions>
                      <v-btn variant="text" :color="color" @click="modal3 = false">Aceptar</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog> -->
              </v-col> 
              
              <v-col col="12" xs="12" md="5" lg="5">Hora termino:</v-col>
              <v-col col="12" xs="12" sm="12" md="6" lg="6">
                <v-text-field :color="color" v-model="form3.horater_estses" label="Indicar hora de termino" :maxlength="5" 
                    variant="outlined" :error-messages="horater_estsesErrors" ref="horater_estses">
                  <template #prepend>
                    <v-dialog ref="dialogRegH" v-model="modal4"  persistent width="350px" overlay-opacity=".1">
                      <template v-slot:activator="{ props }">
                        <v-btn  size="x-large" v-bind="props" density="compact" :color="color"><v-icon>mdi-clock-time-eight-outline</v-icon></v-btn>
                      </template>
                      <v-card class="align-center mx-8">
                        <time-picker :color="color" v-model="form3.horater_estses" automatic></time-picker>
                        <v-card-actions>
                          <v-btn variant="text" :color="color" @click="modal4 = false">Aceptar</v-btn>
                        </v-card-actions>
                      </v-card>
                    </v-dialog>
                  </template>  
                </v-text-field>
                <!-- <v-dialog ref="dialogRegH" v-model="modal4"  persistent width="350px" overlay-opacity=".1">
                  <template v-slot:activator="{ props }">
                    <v-text-field readonly :color="color" v-model="form3.horater_estses" prepend-icon="mdi-clock-time-eight-outline" label="Indicar hora de termino" 
                    variant="outlined" v-bind="props" :error-messages="horater_estsesErrors" ref="horater_estses"></v-text-field>
                  </template>
                  <v-card class="align-center mx-8">
                    <time-picker :color="color" v-model="form3.horater_estses" automatic></time-picker>
                    <v-card-actions>
                      <v-btn variant="text" :color="color" @click="modal4 = false">Aceptar</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog> -->
              </v-col> 
              <v-col col="12" xs="12" sm="12" md="12" lg="12"></v-col>
              <v-col col="12" xs="12" sm="12" md="4" lg="4" class="text-center">
                Descripción:
              </v-col>
              <v-col col="12" xs="12" sm="12" md="7" lg="7">
                <v-textarea type="text" :color="color" v-model="form3.descripcion_estses" :counter="2000" min="1" label="Descripción" variant="outlined" :error-messages="descripcion_estsesErrors" @change="v$.form3.descripcion_estses.$touch()" @blur="v$.form3.descripcion_estses.$touch()" ref="descripcion_estses" maxlength ="2000"></v-textarea>
              </v-col>               
            </v-row>
          </v-container>    
          <!-- </v-virtual-scroll> -->
        </v-card-text>
        <v-card-actions class="d-flex justify-center">
          <v-btn variant="tonal" size="large" color="red" @click="dialogEstado_sesion = false"> Cerrar </v-btn>
          <v-btn variant="tonal" :loading="submitForm" :disabled="submitForm" size="large" :color="color"  @click="validarFormulario"> Guardar </v-btn>
        </v-card-actions>   
    </v-card>
  </v-dialog>
  <v-dialog v-model="dialogReportes" scrollable max-width="60%" content-class="rounded-xl elevation-12" :retain-focus="false">
      <v-card class="rounded-xl">
        <v-img height="90" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
          <v-card title="Reportes de la sesión" color="rgba(0, 0, 0, 0)" class="text-white" variant="flat"></v-card>
        </v-img>
        <v-card-text>
          <!-- <v-virtual-scroll  :items="itemsScroll" height="450" item-height="80"> -->
          <v-container fluid class="mt-6">
            <v-row class="justify-center text-center">
              <v-col cols="auto" v-for="reporte in reportes" :key="reporte.tipo">
                <v-hover v-slot="{ isHovering, props }" open-delay="200">
                    <v-card :elevation="isHovering ? 16: 2" :class="{ 'on-hover': isHovering } " color="#00695c" height="180" width="180"
                    class="text-center justify-center" v-bind="props"  @click="descargar_excel(reporte.tipo, reporte.nombre)">
                        <v-icon size="50px" class="mt-8" > mdi-file-excel</v-icon>
                        <v-spacer></v-spacer>
                        {{ reporte.nombre }}
                    </v-card>
                </v-hover>
              </v-col>
            </v-row>
          </v-container>
          <!-- </v-virtual-scroll> -->
        </v-card-text>
      <v-card-actions class="d-flex justify-center">
        <v-btn variant="tonal" size="large" color="red" @click="dialogReportes = false"> Cerrar </v-btn>
      </v-card-actions> 
    </v-card>
  </v-dialog>
  <v-dialog v-model="dialogActa" scrollable max-width="60%" content-class="rounded-xl elevation-12" :retain-focus="false">
    <v-card class="rounded-xl">
      <v-img height="160" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
          <v-card title="Carga de acta de la sesión" color="rgba(0, 0, 0, 0)"  class="text-white" variant="flat">
              <v-avatar :color="color" size="60">
                  <v-icon large>
                      mdi-file-arrow-up-down-outline
                  </v-icon>
              </v-avatar>
          </v-card>
      </v-img>
      <v-card-text class="justify-center align-center text-center">
        <v-container fluid>
          <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                  Carga el archivo PDF o varios en un ZIP <br> (Si hay un archivo guardado, será reemplazado)
              </v-col>
              <v-col col="12" xs="12" sm="12" md="8" lg="8">
                  <v-file-input v-model="formActaSesion.file" :color="color" counter label="Selecciona el archivo PDF o ZIP" placeholder="Selecciona el archivo" 
                  prepend-icon="mdi-paperclip" variant="outlined" :error-messages="formActaSesionFileErrors" :show-size="1000" accept=".pdf, .zip" :rules="rules">
                      <template v-slot:selection="{ fileNames }">
                          <template v-for="(fileName, index) in fileNames" :key="fileName">
                              <v-chip
                              v-if="index < 2"
                              :color="color"
                              label
                              class="me-2"
                              >
                              {{ fileName }}
                              </v-chip>

                              <span
                              v-else-if="index === 2"
                              class="text-overline text-grey-darken-3 mx-2"
                              >
                              +{{ formActaSesion.file.length - 2 }} Archivo(s)
                              </span>
                          </template>
                      </template>
                  </v-file-input>
              </v-col>

              <v-container fluid>
                  <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                          <v-btn :color="color" :loading="submitForm" :disabled="submitForm" block size="large" @click="validaActaSesion">
                              <v-icon left>
                                mdi-upload
                              </v-icon> Cargar
                          </v-btn>
                      </v-col>
                      <v-col col="12" xs="12" sm="12" md="4" lg="4" v-if="formActaSesion.actaSesion != null">
                          <v-btn :color="color" :href="formActaSesion.actaSesion" download target="_blank" block size="large">
                              <v-icon left>
                                mdi-download
                              </v-icon> Descargar
                          </v-btn>
                      </v-col>
                  </v-row>
              </v-container>
          </v-row>
        </v-container>
      </v-card-text>
      <v-card-actions class="d-flex justify-center">
          <v-btn variant="tonal" size="large" color="red" @click="dialogActa = false"> Cerrar </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
  <v-dialog v-model="dialogDocumentos" scrollable max-width="60%" content-class="rounded-xl elevation-12" :retain-focus="false">
    <v-card class="rounded-xl">
      <v-img height="160" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
          <v-card title="Carga de documento de la sesión" color="rgba(0, 0, 0, 0)"  class="text-white" variant="flat">
              <v-avatar :color="color" size="60">
                  <v-icon large>
                      mdi-file-arrow-up-down-outline
                  </v-icon>
              </v-avatar>
          </v-card>
      </v-img>
      <v-card-text class="justify-center align-center text-center">
        <v-container fluid>
          <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                  Carga el archivo PDF o un comprimido ZIP <br> (Si hay un archivo guardado, será reemplazado)
              </v-col>
              <v-col col="12" xs="12" sm="12" md="8" lg="8">
                  <v-file-input v-model="formDocSesion.file" :color="color" counter label="Selecciona el archivo PDF o ZIP" placeholder="Selecciona el archivo" 
                  prepend-icon="mdi-paperclip" variant="outlined" :error-messages="formDocSesionFileErrors" :show-size="1000" accept=".pdf, .zip" :rules="rules">
                      <template v-slot:selection="{ fileNames }">
                          <template v-for="(fileName, index) in fileNames" :key="fileName">
                              <v-chip
                              v-if="index < 2"
                              :color="color"
                              label
                              class="me-2"
                              >
                              {{ fileName }}
                              </v-chip>

                              <span
                              v-else-if="index === 2"
                              class="text-overline text-grey-darken-3 mx-2"
                              >
                              +{{ formDocSesion.file.length - 2 }} Archivo(s)
                              </span>
                          </template>
                      </template>
                  </v-file-input>
              </v-col>

              <v-container fluid>
                  <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                          <v-btn :color="color" :loading="submitForm" :disabled="submitForm" block size="large" @click="validaDocumentoSesion">
                              <v-icon left>
                                mdi-upload
                              </v-icon> Cargar
                          </v-btn>
                      </v-col>
                      <v-col col="12" xs="12" sm="12" md="4" lg="4" v-if="formDocSesion.documentoSesion != null">
                          <v-btn :color="color" :href="formDocSesion.documentoSesion" download target="_blank" block size="large" >
                              <v-icon left>
                                mdi-download
                              </v-icon> Descargar
                          </v-btn>
                      </v-col>
                  </v-row>
              </v-container>
          </v-row>
        </v-container>
      </v-card-text>
      <v-card-actions class="d-flex justify-center">
          <v-btn variant="tonal" size="large" color="red" @click="dialogDocumentos = false"> Cerrar </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
  <v-dialog v-model="dialogMinutas" scrollable max-width="60%" content-class="rounded-xl elevation-12" :retain-focus="false">
    <v-card class="rounded-xl">
      <v-img height="160" cover src="@/img/fondo_nombre1.png" class="d-flex justify-center align-center text-center">
          <v-card title="Carga de minuta de la sesión" color="rgba(0, 0, 0, 0)"  class="text-white" variant="flat">
              <v-avatar :color="color" size="60">
                  <v-icon large>
                      mdi-file-arrow-up-down-outline
                  </v-icon>
              </v-avatar>
          </v-card>
      </v-img>
      <v-card-text class="justify-center align-center text-center">
        <v-container fluid>
          <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
              <v-col col="12" xs="12" sm="12" md="12" lg="12" class="text-center">
                  Carga el archivo PDF o varios en un ZIP <br> (Si hay un archivo guardado, será reemplazado)
              </v-col>
              <v-col col="12" xs="12" sm="12" md="8" lg="8">
                  <v-file-input v-model="formMinutaSesion.file" :color="color" counter label="Carga la minuta o los archivos en un ZIP" placeholder="Selecciona el archivo" 
                  prepend-icon="mdi-paperclip" variant="outlined" :error-messages="formMinutaSesionFileErrors" :show-size="1000" accept=".pdf, .zip" :rules="rules">
                      <template v-slot:selection="{ fileNames }">
                          <template v-for="(fileName, index) in fileNames" :key="fileName">
                              <v-chip
                              v-if="index < 2"
                              :color="color"
                              label
                              class="me-2"
                              >
                              {{ fileName }}
                              </v-chip>

                              <span
                              v-else-if="index === 2"
                              class="text-overline text-grey-darken-3 mx-2"
                              >
                              +{{ formMinutaSesion.file.length - 2 }} Archivo(s)
                              </span>
                          </template>
                      </template>
                  </v-file-input>
              </v-col>

              <v-container fluid>
                  <v-row class="d-flex flex-md-row flex-column justify-center align-center text-center">
                      <v-col col="12" xs="12" sm="12" md="4" lg="4">
                          <v-btn :color="color" :loading="submitForm" :disabled="submitForm" block size="large"  @click="validaMinutaSesion">
                              <v-icon left>
                                mdi-upload
                              </v-icon> Cargar
                          </v-btn>
                      </v-col>
                      <v-col col="12" xs="12" sm="12" md="4" lg="4" v-if="formMinutaSesion.minutaSesion != null">
                        <v-btn :color="color" :href="formMinutaSesion.minutaSesion" download target="_blank" block size="large">
                            <v-icon left >
                              mdi-download
                            </v-icon> Descargar
                        </v-btn>
                      </v-col>
                  </v-row>
              </v-container>
          </v-row>
        </v-container>
      </v-card-text>
      <v-card-actions class="d-flex justify-center">
          <v-btn variant="tonal" size="large" color="red" @click="dialogMinutas = false"> Cerrar </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
  <v-snackbar v-model="snackbar" vertical :timeout="1500" :color="colorSnackbar" location="center"  elevation="24">
    <div class="text-subtitle-1 "><v-icon left>mdi-information-outline</v-icon> {{ textoTituloSnackbar }}</div>
    <p>{{ textoSnackbar }}</p>
    <div v-if="errors.length != 0" >
      <p v-for="error in errors" :key="error">
        {{ error }}
      </p>
    </div>
    <!-- <template v-slot:actions>
      <v-btn variant="text" @click="snackbar = false">
        Cerrar
      </v-btn>
    </template> -->
  </v-snackbar>
</template>

<script lang="ts">
    import "@fontsource/montserrat/600.css";
    import { useSisecedStore } from '@/store/sisecedStore';
    import moment from 'moment';
    import { TimePicker } from "vue-material-time-picker";
    import "vue-material-time-picker/dist/style.css";
    import { ReportesC, Integrantes } from '@/models/types'
    import { useVuelidate } from '@vuelidate/core'
    import { required, maxLength, helpers  } from '@vuelidate/validators'    

    moment.locale('es',{
      invalidDate: moment().format('DD-MM-YYYY')
    });    

    export default {
      emits: ["tengo_resultados"],
      components: {
            TimePicker
      },
      setup(){
        return{
          v$: useVuelidate()
        }
      },        
      data: ()=>({
        submitForm: false,
        color: '',
        snackbar: false,
        colorSnackbar: '',
        textoTituloSnackbar: '',
        textoSnackbar: '',
        errors: <any>[],
        headers: <any>[
          {title: 'Tipo de sesión', align: 'center', key: 'tipoSesionNombre'},
          {title: 'Fecha', align: 'center', key: 'fechaInicio'},
          {title: 'Hora programada de inicio', align: 'center', key: 'horaInicio'},     
          {title: 'Catálogo de integrantes', align: 'center', key: 'cat_integrantes'},
          {title: 'Inicio de sesión', align: 'center', key: 'inicio_sesion'},
          {title: 'Fin de sesión', align: 'center', key: 'fin_sesion'},
          {title: 'Seguimiento al orden del día', align: 'center', key: 'seg_orden_dia'},
          {title: 'Estado de sesión ', align: 'center', key: 'estado_sesion'},
          //{title: 'Reportes', align: 'center', key: 'reportes'},
          //{title: 'Documentos de la sesión', align: 'center', key: 'doc_sesion'},
          //{title: 'Acta', align: 'center', key: 'acta'},
          //{title: 'Minuta', align: 'center', key: 'minuta'},
        ],
        page: 1,
        itemsPerPage:50,
        desserts: [],
        sesion: null,
        search: '',
        select_sesion: [],
        fechaForms: '',
        form1:{
          fechaProgramada: '',
          horaProgramada: '',
          fecha: <any>'',
          hora: '',
          domicilio: '',
          radio: false,
          prensa: false,
          television: false,
          observaciones: ''
        },
        form1Default:{
          fechaProgramada: '',
          horaProgramada: '',
          fecha: <any>'',
          hora: '',
          domicilio: '',
          radio: false,
          prensa: false,
          television: false,
          observaciones: ''
        },
        //Integrantes Consejo
        desserts2: [],       
        //Partidos 
        dialogInicioSesion: false,
        dialogFinSesion: false,
        dialogEstado_sesion: false,
        modal: false,
        dialogRegCat: false,
        dialogReportes: false,
        modal2: false, 
        itemsScroll: Array.from({ length: 1 }, (k, v) => v + 1),
        form3:{
          Estado_sesion: null,
          horaini_estses: '',
          horater_estses: '',
          descripcion_estses: '',
        },
        form3Default:{
          Estado_sesion: null,
          horaini_estses: '',
          horater_estses: '',
          descripcion_estses: '',
        },
        estadosSesiones: [
          {
            name: 'Sin Quórum',
            value: 1,
          },
          {
            name: 'Segunda Convocatoria',
            value: 2,
          },
          {
            name: 'En receso',
            value: 3,
          },
          {
            name: 'Concluyó receso',
            value: 4,
          },
          {
            name: 'Suspendida',
            value: 5,
          },
          {
            name: 'Reanudada',
            value: 6,
          },
          {
            name: 'Prolongada',
            value: 7,
          },
        ],
        modal3: false,
        modal4: false,
        rulesObligatory:[
          (v: string) => !!v || 'Este campo es obligatorio',
          (v: string) => (v && v.length <= 150) || 'No pueden ser más de 150 caracteres'
        ],
        rulesObligatoryRadio:[
          (v: string) => !!v || 'Este campo es obligatorio',
        ],
        dialogIntegrantesConsejo: false,
        tab2: 'tab-1',
        dessertsIntegrantes: <Array<Integrantes>>[],
        dessertsPartidos: <Array<Integrantes>>[],
        dessertsIndependientes: <Array<Integrantes>>[],
        idSesionEdicion: 0,
        reportes: <Array<ReportesC>>[
          {
              nombre: 'Inicio y fin de sesión',
              tipo: 1,
              link: ''
              
          },
          {
              nombre: 'Sentido del voto',
              tipo: 2,
              link: ''
          },
          {
              nombre: 'Incidentes',
              tipo: 3,
              link: ''
          },
          {
              nombre: 'Intervenciones',
              tipo: 4,
              link: ''
          },
          {
              nombre: 'Estados de la sesión',
              tipo: 5,
              link: ''
          },
          {
              nombre: 'Integrantes del consejo',
              tipo: 6,
              link: ''
          },
          {
              nombre: 'Representanciones de Partidos Políticos y de Candidaturas sin Partido',
              tipo: 7,
              link: ''
          },
        ],
        idSesionDialogs: 0,
        dialogOrdenDDia: false,
        dialogAvisoVota: false,
        contadorAsistenciaIntegrantes: 0,
        contadorVotacionIntegrantes: 0,
        headers3: <any>[
          {title: 'Punto', align: 'center', key: 'numPunto', sortable: true},
          {title: 'Descripción', align: 'center', key: 'descripcion', sortable: true},
          {title: 'Votación', align: 'center', key: 'votaciones', sortable: false},     
          {title: 'Intervención', align: 'center', key: 'intervenciones', sortable: false},
          {title: 'Incidente', align: 'center', key: 'incidentes', sortable: false},
          {title: 'Inhabilita', align: 'center', key: 'quitarlos', sortable: false}
        ],
        desserts3:[],
        pasoPuntos: 'tab-1',
        itemsPerPageO: 10,
        buscar2: '',
        pageO: 1,
        dialogQuitaPunto: false,
        dialogAgregaQPunto: false,
        numIntervencion: 1,
        numIncidente: 1,
        form2: {
          numPunto: '',
          descripcion:'',
          replica: ''
        },
        form2Default: {
          numPunto: '',
          descripcion:'',
          replica: ''
        },
        form2Edited: {
          descripcion:'',
          numIntervencion: 1,
          replica: ''
        },
        form2EditedDefault: {
          descripcion:'',
          numIntervencion: 1,
          replica: ''
        },
        form31: {
          numPunto: '',
          descripcion:'',
          replica: ''
        },
        form31Default: {
          numPunto: '',
          descripcion:'',
          replica: ''
        },
        form31Edited: {
          descripcion:'',
          numIncidente: 1,
          replica: ''
        },
        form31EditedDefault: {
          descripcion:'',
          numIncidente: 1,
          replica: ''
        },
        pageInc: 1,
        
        headersIntervencion: <any>[
          {title: 'No.', align: 'center', key: 'idIntervencion', sortable: true},
          {title: 'Descripcion intervenciones', align: 'center', key: 'descripcionEvento', sortable: true},
          {title: 'Réplica', align: 'center', key: 'replica', sortable: false},     
          {title: 'Editar', align: 'center', key: 'editar', sortable: false},
          {title: 'Eliminar', align: 'center', key: 'quitar', sortable: false}
        ],
        dessertsIntervencion:[],
        pageInt:1,
        itemsPerPageInt: 3,
        headersIncidente: <any>[
          {title: 'No.', align: 'center', key: 'idIncidente', sortable: true},
          {title: 'Descripcion incidentes', align: 'center', key: 'descripcionEvento', sortable: true},
          {title: 'Réplica', align: 'center', key: 'replica', sortable: false},     
          {title: 'Editar', align: 'center', key: 'editar', sortable: false},
          {title: 'Eliminar', align: 'center', key: 'quitar', sortable: false}
        ],
        dessertsIncidente:[],
        itemsPerPageInc: 6,
        dialogDocumentos: false,
        dialogActa: false,
        dialogMinutas: false,
        fileActa: <Array<File>>[],
        fileDoc: <Array<File>>[],
        fileMinuta: <Array<File>>[],
        formVotacion:{
          observaciones: ''
        },
        formVotacionDefault:{
          observaciones: ''
        },
        idPuntoVotacion: '',
        idIntervencionesEdited : 0,
        dialogQuitaIntervencion: false,
        idIncidenteEdited : 0,
        dialogQuitaIncidente: false,
        formDocSesion: {
          idSesion: 0,
          file: <Array<File>>[],
          documentoSesion: ''
        },
        formDocSesionDefault: {
          idSesion: 0,
          file: <Array<File>>[],
          documentoSesion: ''
        },
        formActaSesion: {
          idSesion: 0,
          file: <Array<File>>[],
          actaSesion: ''
        },
        formActaSesionDefault: {
          idSesion: 0,
          file: <Array<File>>[],
          actaSesion: ''
        },
        formMinutaSesion: {
          idSesion: 0,
          file: <Array<File>>[],
          minutaSesion: ''
        },
        formMinutaSesionDefault: {
          idSesion: 0,
          file: <Array<File>>[],
          minutaSesion: ''
        },
        rules: [
          (value: string | any[]) => {
            return !value || !value.length || value[0].size < 10000000 || 'El Archivo no puede ser mayor a 10 MB!'
          },
        ],
      }),
      validations(){
        return{
          form3:{
            Estado_sesion: {required},
            horaini_estses: {required},
            horater_estses: {required},
            descripcion_estses: {required}
          },
          form1:{
            //fecha: {required},
            hora: {required},
            domicilio: {required},
            observaciones: {required}
          },
          form2:{
            descripcion: {required},
            replica: {required}
          },
          form2Edited:{
            descripcion: {required},
            replica: {required}
          },
          form31:{
            descripcion: {required},
            replica: {required}
          },
          form31Edited:{
            descripcion: {required},
            replica: {required}
          },
          formVotacion:{
            observaciones: {required}
          },
          formDocSesion: {
            file: {required}
          },
          formActaSesion: {
            file: {required}
          },
          formMinutaSesion: {
            file: {required}
          },
        }
      }, 
      computed:{ 
        Estado_sesionErrors(){
           const errors: any = [];
          if(!this.v$.form3.Estado_sesion.$error) return errors
            this.v$.form3.Estado_sesion.required &&errors.push('Indicar el estado de la sesión')
            return errors
        },      
        horaini_estsesErrors(){
           const errors: any = [];
          if(!this.v$.form3.horaini_estses.$error) return errors
            this.v$.form3.horaini_estses.required &&errors.push('Indicar la hora de la sesión')
            return errors
        },   
        horater_estsesErrors(){
           const errors: any = [];
          if(!this.v$.form3.horater_estses.$error) return errors
            this.v$.form3.horater_estses.required &&errors.push('Indicar la hora de termino')
            return errors
        },
        descripcion_estsesErrors(){
           const errors: any = [];
          if(!this.v$.form3.descripcion_estses.$error) return errors
            this.v$.form3.descripcion_estses.required &&errors.push('Indicar una descripción')
            return errors
        },         
        fechaErrors(){
          const errors: any = [];
          if(!this.v$.form1.fecha.$error) return errors
            this.v$.form1.fecha.required &&errors.push('Indicar la fecha de inicio de la sesión')
            return errors
        },      
        horaErrors(){
           const errors: any = [];
          if(!this.v$.form1.hora.$error) return errors
            this.v$.form1.hora.required &&errors.push('Indicar la hora de la sesión')
            return errors
        },   
        domicilioErrors(){
           const errors: any = [];
          if(!this.v$.form1.domicilio.$error) return errors
            this.v$.form1.domicilio.required &&errors.push('Indicar el domicilio del inicio de la sesión')
            return errors
        }, 
        observacionesErrors(){
           const errors: any = [];
          if(!this.v$.form1.observaciones.$error) return errors
            this.v$.form1.observaciones.required &&errors.push('Indicar si hay observaciones en el inicio de la sesión')
            return errors
        },  
        descripcionErrors(){
           const errors: any = [];
          if(!this.v$.form2.descripcion.$error) return errors
            this.v$.form2.descripcion.required &&errors.push('Es obligatorio llenar el campo de la descripción')
            return errors
        },
        replicaErrors(){
           const errors: any = [];
          if(!this.v$.form2.replica.$error) return errors
            this.v$.form2.replica.required &&errors.push('Es obligatorio llenar el campo de la réplica')
            return errors
        },  
        descripcionEditedErrors(){
           const errors: any = [];
          if(!this.v$.form2Edited.descripcion.$error) return errors
            this.v$.form2Edited.descripcion.required &&errors.push('Es obligatorio llenar el campo de la descripción')
            return errors
        },
        replicaEditedErrors(){
           const errors: any = [];
          if(!this.v$.form2Edited.replica.$error) return errors
            this.v$.form2Edited.replica.required &&errors.push('Es obligatorio llenar el campo de la réplica')
            return errors
        },  
        descripcion3Errors(){
           const errors: any = [];
          if(!this.v$.form31.descripcion.$error) return errors
            this.v$.form31.descripcion.required &&errors.push('Es obligatorio llenar el campo de la descripción')
            return errors
        },
        replica3Errors(){
           const errors: any = [];
          if(!this.v$.form31.replica.$error) return errors
            this.v$.form31.replica.required &&errors.push('Es obligatorio llenar el campo de la réplica')
            return errors
        },
        observacionesVotacionErrors(){
           const errors: any = [];
          if(!this.v$.formVotacion.observaciones.$error) return errors
            this.v$.formVotacion.observaciones.required &&errors.push('Indicar si hay observaciones, o en su caso, la leyenda sin observaciones')
            return errors
        },
        formDocSesionFileErrors(){
          const errors: any = [];
          if(!this.v$.formDocSesion.file.$error) return errors
            this.v$.formDocSesion.file.required&&errors.push('Indicar el documento de la sesión')
            return errors        
        },
        formActaSesionFileErrors(){
          const errors: any = [];
          if(!this.v$.formActaSesion.file.$error) return errors
            this.v$.formActaSesion.file.required&&errors.push('Indicar el acta de la sesión')
            return errors        
        },
        formMinutaSesionFileErrors(){
          const errors: any = [];
          if(!this.v$.formMinutaSesion.file.$error) return errors
            this.v$.formMinutaSesion.file.required&&errors.push('Indicar la minuta de la sesión')
            return errors        
        },
        pageCount () {
            return Math.ceil(this.desserts.length / this.itemsPerPage)
        },
        pageCountO () {
          return Math.ceil(this.desserts3.length / this.itemsPerPageO)
        },
        pageCountInt () {
          return Math.ceil(this.dessertsIntervencion.length / this.itemsPerPageInt)
        },
        pageCountInc () {
          return Math.ceil(this.dessertsIncidente.length / this.itemsPerPageInc)
        },
      },
      methods:{
        coloresItem(item: number){
          if ( item <= 4 ){
            return 'red'
          } else if ( item == 5 ){
            return 'orange-darken-1'
          } else if ( item == 6 ){
            return '#FFD740'
          } else if ( item >= 7 ){
            return 'green'
          } 
        },
        snackBarFaltantes(){
          this.errors = [];
          this.errors.push('Faltan puntos con votación requerida por realizar la votación');
          this.textoTituloSnackbar = 'Atención';
          this.colorSnackbar = 'red-darken-2';
          this.snackbar = true;
        },
        async validarFormulario () {
          this.v$.$validate();
          if (!this.v$.form3.$invalid) {
            const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
            }
            let postId = {
              idSesion: this.idSesionEdicion,
              estadoSesion: this.form3.Estado_sesion,
              horaInicio: this.form3.horaini_estses,
              horaFin: this.form3.horater_estses,
              descripcion: this.form3.descripcion_estses
            };
            this.submitForm = true;
            const Response = await this.$axios.post('/log-sesiones/agregar', postId, { headers: headers })
            .then( function(response){
              console.log(response.data);
              return response.data;
            }).catch( function(error){
              let login = error.response.data.login;
              if ( login == false ){
                  return false
              } else {
                  return error;
              }
            });
            if ( Response == false ){
                this.abrirPadre();
            } else {
              this.errors = [];
              if ( Response.ok == true ){
                this.textoTituloSnackbar = 'Correcto';
                this.textoSnackbar = Response.msg;
                this.colorSnackbar = this.color;
                this.snackbar = true;
                this.dialogEstado_sesion = false;
                this.form3 = Object.assign({}, this.form3Default);
                this.v$.$reset();
                this.submitForm = false;
              } else {
                console.log(Response);
                this.textoTituloSnackbar = 'Error';
                this.textoSnackbar = Response.msg;
                this.colorSnackbar = 'red-darken-2';
                this.snackbar = true;
                this.dialogEstado_sesion = false;
                this.form3 = Object.assign({}, this.form3Default);
                this.v$.$reset();
                this.submitForm = false;
              }
            }
          } else{
            this.errors = [];
            if(!this.form3.Estado_sesion){
              this.errors.push('Indicar el estado de la sesión.');
              (<any>this.$refs.Estado_sesion).focus();
            } else if (!this.form3.horaini_estses){
              this.errors.push('Indicar la hora de inicio.');
              (<any>this.$refs.horaini_estses).focus();
            } else if (!this.form3.horater_estses){
              this.errors.push('Indicar la hora de termino.');
              (<any>this.$refs.horater_estses).focus();
            } else if (!this.form3.descripcion_estses){
              this.errors.push('Indicar la hora de termino.');
              (<any>this.$refs.descripcion_estses).focus();
            }
          }
        },
        abreDialogIntegrantes(item: any){
          this.idSesionEdicion = item.idSesion;
          this.consultaIntegrantes(item.idSesion);
          this.consultaIntegrantesPartidos(item.idSesion);
          this.consultaIndepPartidos(item.idSesion);
          this.dialogIntegrantesConsejo = true;
        },
        inicio_sesion(item: any){
          console.log('hola 2');
          console.log(item);
          this.dessertsIntegrantes = [];
          this.dessertsPartidos = [];
          this.dessertsIndependientes = [];
          this.form1.fecha = item.fechaInicio;
          this.form1.hora = item.horaInicio;
          this.form1.horaProgramada = item.horaInicio;
          // this.fechaForms = moment(item.fechaInicio).format('YYYY-MM-DD');
          this.fechaForms = item.fechaInicioBase;
          
          console.log(item.fechaInicioBase);
          this.idSesionEdicion = item.idSesion;
          this.consultaIntegrantesInicioSesion(item.idSesion);
          this.consultaIntegrantesPartidosInicioSesion(item.idSesion);
          this.consultaIndepPartidosInicioSesion(item.idSesion);
          this.dialogInicioSesion = true;
        },
        async consulta_inicio_sesion(item: any){
          console.log(item.fechaInicio);
          this.dessertsIntegrantes = [];
          this.dessertsPartidos = [];
          this.dessertsIndependientes = [];
          
          this.idSesionEdicion = item.idSesion;

          const headers = {
            'Content-Type': 'application/json',
            'x-token-admin':localStorage.jwt
          }
          let postId = {
            idSesion: this.idSesionEdicion
          };
          const Response = await this.$axios.post('/asistencia/consulta-asistencia-inicio', postId, { headers: headers })
          .then( function(response){
            console.log(response.data);
            return response.data;
          }).catch( function(error){
            let login = error.response.data.login;
            if ( login == false ){
                return false
            } else {
                return error;
            }
          });
          if ( Response == false ){
              this.abrirPadre();
          } else {
            console.table(Response.asistenciaDB[0]);
            console.table(Response.asistenciaConsejoDB[0]);
            console.table(Response.asistenciaPartidosDB[0]);
            console.table(Response.asistenciaPartidosIndepDB[0]);
            this.form1 = Object.assign({}, Response.asistenciaDB[0][0]);
            this.form1.horaProgramada = item.horaInicio;
            this.form1.fecha = item.fechaInicioBase;
            this.fechaForms = moment(item.fechaInicioBase).format('YYYY-MM-DD');
            //this.form1.hora = Response.asistenciaDB[0].hora;
            console.log('Este es form1');
            console.table(this.form1);
            this.dessertsIntegrantes = Response.asistenciaConsejoDB[0];
            this.dessertsPartidos = Response.asistenciaPartidosDB[0];
            this.dessertsIndependientes = Response.asistenciaPartidosIndepDB[0];
            this.dialogInicioSesion = true;
            console.log('Este es form1 después');
            console.table(this.form1);
            //console.table(this.dessertsIntegrantes);
          }
        },
        fin_sesion(item: any){
          console.log('hola');
          console.log(item);
          this.form1.fecha = item.fechaInicio;
          this.fechaForms = moment(item.fechaInicio).format('YYYY-MM-DD');
          console.log(this.fechaForms);
          this.idSesionEdicion = item.idSesion;
          this.consultaIntegrantesInicioSesion(item.idSesion);
          this.consultaIntegrantesPartidosInicioSesion(item.idSesion);
          this.consultaIndepPartidosInicioSesion(item.idSesion);
          this.dialogFinSesion = true;
        },
        estado_sesion(item: any){
          this.idSesionEdicion = item.idSesion;
          this.dialogEstado_sesion = true;
        },        
        cierraDialogEstado_sesion(){
          this.idSesionEdicion = 0;
          this.v$.$reset();
          this.form3 = Object.assign({}, this.form3Default);
        },
        updateDate(val : any) {
          this.form1.fecha = moment(val).format('DD-MM-YYYY');
          this.fechaForms = moment(val).format('YYYY-MM-DD');
          console.log(val)
        },
        reportesDialogs(item: number){
          this.idSesionDialogs = item;
          this.dialogReportes = true;
        },
        abrirPadre() {
            this.$emit('tengo_resultados', true);
        },
        async consultaIntegrantes(item: number){
          console.log(item);
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: item
          };
          const local = await this.$axios.post('/asistencia/consulta-asistencia-consejo', postId, { headers: headers })
          .then( function(response){
              console.table(response.data.AsistenciaDB[0]);
              return response.data.AsistenciaDB[0];
          }).catch( function(error){
              let login = error.response.data.login;
              if ( login == false ){
                  return false
              } else {
                  return error;
              }
          });
          if ( local == false ){
              this.abrirPadre();
          } else {
              this.dessertsIntegrantes = local;
              //console.table(this.dessertsIntegrantes);
          }
      },
      async consultaIntegrantesInicioSesion(item: number){
          console.log(item);
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: item
          };
          const local = await this.$axios.post('/asistencia/consulta-asistencia-consejo', postId, { headers: headers })
          .then( function(response){
              console.table(response.data.AsistenciaDB[0]);
              return response.data.AsistenciaDB[0];
          }).catch( function(error){
              let login = error.response.data.login;
              if ( login == false ){
                  return false
              } else {
                  return error;
              }
          });
          if ( local == false ){
              this.abrirPadre();
          } else {
            this.dessertsIntegrantes = local;
            //console.table(this.dessertsIntegrantes);
          }
      },
      async consultaIntegrantesPartidos(item: any){
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: item
          };
          const local = await this.$axios.post('/asistencia/consulta-asistencia-partidos', postId, { headers: headers })
              .then( function(response){
                  console.table(response.data.PartidosDB[0]);
                  return response.data.PartidosDB[0];
              }).catch( function(error){
                  console.log(error);
                  let login = error.response.data.login;
                  if ( login == false ){
                      return false
                  } else {
                      return error;
                  }
              });
              if ( local == false ){
                  this.abrirPadre();
              } else {
              if (local.ok != false ){
                  this.dessertsPartidos = local;
              }
            }
        }, 
        async consultaIntegrantesPartidosInicioSesion(item: any){
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: item
          };
          const local = await this.$axios.post('/asistencia/consulta-asistencia-inicio-partidos', postId, { headers: headers })
            .then( function(response){
                console.table(response.data.PartidosDB[0]);
                return response.data.PartidosDB[0];
            }).catch( function(error){
                console.log(error);
                let login = error.response.data.login;
                if ( login == false ){
                    return false
                } else {
                    return error;
                }
            });
            if ( local == false ){
                this.abrirPadre();
            } else {
            if (local.ok != false ){
              this.dessertsPartidos = local;
            }
          }
      }, 
      async consultaIndepPartidos(item: any){
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: item
          };
          const local = await this.$axios.post('/asistencia/consulta-asistencia-partidos-indep', postId, { headers: headers })
              .then( function(response){
                  console.table(response.data.PartidosDB[0]);
                  return response.data.PartidosDB[0];
              }).catch( function(error){
                  console.log(error);
                  let login = error.response.data.login;
                  if ( login == false ){
                      return false
                  } else {
                      return error;
                  }
              });
              if ( local == false ){
                  this.abrirPadre();
              } else {
              if (local.ok != false ){
                this.dessertsIndependientes = local;
              }
          }
      }, 
      async consultaIndepPartidosInicioSesion(item: any){
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: item
          };
          const local = await this.$axios.post('/asistencia/consulta-asistencia-inicio-partidos-indep', postId, { headers: headers })
              .then( function(response){
                  console.table(response.data.PartidosDB[0]);
                  return response.data.PartidosDB[0];
              }).catch( function(error){
                  console.log(error);
                  let login = error.response.data.login;
                  if ( login == false ){
                      return false
                  } else {
                      return error;
                  }
              });
              if ( local == false ){
                  this.abrirPadre();
              } else {
              if (local.ok != false ){
                this.dessertsIndependientes = local;
                console.table(this.dessertsIndependientes);
                // this.dessertsIndependientes.forEach((element)=>{
                //   element.asistencia = null;
                // })
              }
          }
      }, 
      async validaConsejo(){
        const { valid } = await (<any>this.$refs.formConsejo).validate();
        if ( valid ){
          //console.table(this.dessertsIntegrantes);
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
            datosIntegrantes: this.dessertsIntegrantes,
            idSesion: this.idSesionEdicion
          };
          this.submitForm = true;
          const Response = await this.$axios.post('/asistencia/actualiza-integrantes-consejo-d', postId, { headers: headers })
              .then( function(response){
                  console.log(response.data);
                  return response.data;
              }).catch( function(error){
                  console.log('Error: ', error);
                  let login = error.response.data.login;
                  if ( login == false ){
                      return false
                  } else {
                      return error;
                  }
              });
          if ( Response == false ){
              this.abrirPadre();
          } else{
            this.errors = [];
              if ( Response.ok == true ){
                this.dialogIntegrantesConsejo = false;
                this.textoTituloSnackbar = 'Correcto';
                this.textoSnackbar = Response.msg;
                this.colorSnackbar = this.color;
                this.snackbar = true;
                this.submitForm = false;
              } else {
                this.dialogIntegrantesConsejo = false;
                this.textoTituloSnackbar = 'Error';
                this.textoSnackbar = Response.msg;
                this.colorSnackbar = 'red-darken-2';
                this.snackbar = true;
                this.submitForm = false;
              }
          }
        } else {
          console.log('incompleto consejo');
        }
      },
      async validaPartidos(){
          const { valid } = await (<any>this.$refs.formPartidos).validate();
          if ( valid ){
            const headers = {
                'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
            }
            let postId = {
              datosIntegrantes: this.dessertsPartidos,
              idSesion: this.idSesionEdicion
            };
            this.submitForm = true;
            const Response = await this.$axios.post('/asistencia/actualiza-integrantes-partidos-d', postId, { headers: headers })
                .then( function(response){
                    console.log(response.data);
                    return response.data;
                }).catch( function(error){
                    console.log('Error: ', error);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
            if ( Response == false ){
                this.abrirPadre();
            } else{
              this.errors = [];
                if ( Response.ok == true ){
                  this.dialogIntegrantesConsejo = false;
                  this.textoTituloSnackbar = 'Correcto';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.snackbar = true;
                  this.submitForm = false;
                } else {
                  this.dialogIntegrantesConsejo = false;
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.snackbar = true;
                  this.submitForm = false;
                }
            }
          } else {
            console.log('incompleto');
          }
      },
      async validaIndependientes(){
        const { valid } = await (<any>this.$refs.formIndependientes).validate();
        if ( valid ){
          const headers = {
                'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
            }
            let postId = {
              datosIntegrantes: this.dessertsIndependientes,
              idSesion: this.idSesionEdicion
            };
            this.submitForm = true;
            const Response = await this.$axios.post('/asistencia/actualiza-integrantes-partidos-d', postId, { headers: headers })
                .then( function(response){
                    console.log(response.data);
                    return response.data;
                }).catch( function(error){
                    console.log('Error: ', error);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
            if ( Response == false ){
                this.abrirPadre();
            } else{
              this.errors = [];
                if ( Response.ok == true ){
                  this.dialogIntegrantesConsejo = false;
                  this.textoTituloSnackbar = 'Correcto';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.snackbar = true;
                  this.submitForm = false;
                } else {
                  this.dialogIntegrantesConsejo = false;
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.snackbar = true;
                  this.submitForm = false;
                }
            }
        } else {
          console.log('incompleto partidos');
        }
      },
      async listadoSesiones(){
        const headers = {
          'Content-Type': 'application/json',
          'x-token-admin':localStorage.jwt
        }
        const local = await this.$axios.get('/nueva-sesion/consulta-sesiones-d', { headers: headers })
        .then( function(response){
          return response.data;
        }).catch( function(error){
          console.log(error.response.data.login);
          let login = error.response.data.login;
          if ( login == false ){
              return false
          } else {
              return error;
          }
        });
        if ( local == false ){
            this.abrirPadre();
        } else {
          if (local.ok != false ){
            this.select_sesion = local.sesionesDB;
            console.log(this.select_sesion);
          }
        }
      }, 
      async llenaTabla(){
        console.log('hola');
        if ( this.sesion != null ){
          console.log('entre');
          const headers = {
            'Content-Type': 'application/json',
            'x-token-admin':localStorage.jwt
          }
          let postId = {
            idSesion: this.sesion,
          };
          const local = await this.$axios.post('/nueva-sesion/consulta-sesion', postId, { headers: headers })
          .then( function(response){
            return response.data.sesionesDB[0];
          }).catch( function(error){
            let login = error.response.data.login;
            if ( login == false ){
                return false
            } else {
                return error;
            }
          });
          if ( local == false ){
              this.abrirPadre();
          } else {
              this.desserts = local;
              console.table(this.desserts);
          }
        } else {
          console.log('se limpio');
        }
      },
      async consultaPuntosSesion(item: number){
        console.log(item);
        const headers = {
            'Content-Type': 'application/json',
            'x-token-admin':localStorage.jwt
        }
        let postId = {
            idSesion: item
        };
        const Response = await this.$axios.post('/puntos-orden-dia/consulta/', postId, { headers: headers })
            .then( function(response){
                console.log(response.data);
                return response.data;
            }).catch( function(error){
                console.log('Error: ', error);
                let login = error.response.data.login;
                if ( login == false ){
                    return false
                } else {
                    return error;
                }
            });
        if ( Response == false ){
            this.abrirPadre();
        } else{
            if ( Response.ok == true ){
                this.errors = [];
                this.desserts3 = Response.PuntosODD;
            } else {
                this.errors = [];
                this.textoTituloSnackbar = 'Error';
                this.textoSnackbar = Response.response.data.msg;
                this.colorSnackbar = 'red';
                //this.snackbar = true;
            }
        }
      },
      seg_orden_dia (item: any){
        console.log(item);
        this.contadorAsistenciaIntegrantes = item.asistencias;
        this.idSesionEdicion = item.idSesion;
        this.dialogOrdenDDia = true;
        this.consultaPuntosSesion(item.idSesion);
        this.consultaIntegrantesInicioSesion(item.idSesion);
        this.consultaIntegrantesPartidosInicioSesion(item.idSesion);
        this.consultaIndepPartidosInicioSesion(item.idSesion);
        console.log(this.dessertsIndependientes);
      },
      cierraDialogConsejo(){
        this.tab2 = 'tab-1';
        this.idSesionEdicion = 0;
        this.dessertsIntegrantes = [];
        this.dessertsIndependientes = [];
        this.dessertsPartidos = [];
        this.dialogIntegrantesConsejo = false;
      },
      async descargar_excel(tipo:number, nombre:any){
        console.log(tipo);
        console.log(this.idSesionDialogs);
        const headers = {
            'x-token-admin':localStorage.jwt
        }
        let postId = {
          idSesion: this.idSesionDialogs
        };
        if(tipo == 1){
            await this.$axios.post('/nueva-sesion/genera_rpt_InicioFinDistrital', postId, { headers: headers, responseType: 'blob' })
                .then( function(response){
                    //console.table(response.data.integrantesPartidosDB[0]);
                    console.log(response);
                    const blob = new Blob([response.data], {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
                    const url = window.URL.createObjectURL(blob);

                    const a = document.createElement('a');
                    
                    a.href = url;
                    a.download = nombre;
                    a.click();
                    //console.log(a)
                    window.URL.revokeObjectURL(url);
                    //return response.data;
                }).catch( function(error){
                    // console.log(error.response.data.login);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
                // if ( local == false ){
                //     this.abrirPadre();
                // } else {
                // if (local.ok != false ){
                //     //this.dessertsIndependientes = local.integrantesPartidosDB[0];
                // }
            } 
            else if(tipo == 2){
              await this.$axios.post('/nueva-sesion/genera_rpt_SentidoVoto', postId, { headers: headers, responseType: 'blob' })
                .then( function(response){
                    //console.table(response.data.integrantesPartidosDB[0]);
                    console.log(response);
                    const blob = new Blob([response.data], {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
                    const url = window.URL.createObjectURL(blob);

                    const a = document.createElement('a');
                    
                    a.href = url;
                    a.download = nombre;
                    a.click();
                    //console.log(a)
                    window.URL.revokeObjectURL(url);
                    //return response.data;
                }).catch( function(error){
                    // console.log(error.response.data.login);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
                // if ( local == false ){
                //     this.abrirPadre();
                // } else {
                // if (local.ok != false ){
                //     //this.dessertsIndependientes = local.integrantesPartidosDB[0];
                // }
            }                     
            else if(tipo == 3){
            await this.$axios.post('/nueva-sesion/genera_rpt_IncidentesSeg',  postId, { headers: headers, responseType: 'blob' })
                .then( function(response){
                    //console.table(response.data.integrantesPartidosDB[0]);
                    console.log(response);
                    const blob = new Blob([response.data], {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
                    const url = window.URL.createObjectURL(blob);

                    const a = document.createElement('a');
                    
                    a.href = url;
                    a.download = nombre;
                    a.click();
                    //console.log(a)
                    window.URL.revokeObjectURL(url);
                    //return response.data;
                }).catch( function(error){
                    // console.log(error.response.data.login);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
                // if ( local == false ){
                //     this.abrirPadre();
                // } else {
                // if (local.ok != false ){
                //     //this.dessertsIndependientes = local.integrantesPartidosDB[0];
                // }
            }                     
            else if(tipo == 4){
            await this.$axios.post('/nueva-sesion/genera_rpt_IntervencionesSeg', postId, { headers: headers, responseType: 'blob' })
                .then( function(response){
                    //console.table(response.data.integrantesPartidosDB[0]);
                    console.log(response);
                    const blob = new Blob([response.data], {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
                    const url = window.URL.createObjectURL(blob);

                    const a = document.createElement('a');
                    
                    a.href = url;
                    a.download = nombre;
                    a.click();
                    //console.log(a)
                    window.URL.revokeObjectURL(url);
                    //return response.data;
                }).catch( function(error){
                    // console.log(error.response.data.login);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
                // if ( local == false ){
                //     this.abrirPadre();
                // } else {
                // if (local.ok != false ){
                //     //this.dessertsIndependientes = local.integrantesPartidosDB[0];
                // }
            }                     
            else if(tipo == 5){
            await this.$axios.post('/nueva-sesion/genera_rpt_EstadosSesionSeg', postId, { headers: headers, responseType: 'blob' })
                .then( function(response){
                    //console.table(response.data.integrantesPartidosDB[0]);
                    console.log(response);
                    const blob = new Blob([response.data], {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
                    const url = window.URL.createObjectURL(blob);

                    const a = document.createElement('a');
                    
                    a.href = url;
                    a.download = nombre;
                    a.click();
                    //console.log(a)
                    window.URL.revokeObjectURL(url);
                    //return response.data;
                }).catch( function(error){
                    // console.log(error.response.data.login);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
                // if ( local == false ){
                //     this.abrirPadre();
                // } else {
                // if (local.ok != false ){
                //     //this.dessertsIndependientes = local.integrantesPartidosDB[0];
                // }
            }                     
            else if(tipo == 6){
              await this.$axios.post('/nueva-sesion/genera_rpt_IntegrantesSeg', postId, { headers: headers, responseType: 'blob' })
                .then( function(response){
                    //console.table(response.data.integrantesPartidosDB[0]);
                    console.log(response);
                    const blob = new Blob([response.data], {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
                    const url = window.URL.createObjectURL(blob);

                    const a = document.createElement('a');
                    
                    a.href = url;
                    a.download = nombre;
                    a.click();
                    //console.log(a)
                    window.URL.revokeObjectURL(url);
                    //return response.data;
                }).catch( function(error){
                    // console.log(error.response.data.login);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
                // if ( local == false ){
                //     this.abrirPadre();
                // } else {
                // if (local.ok != false ){
                //     //this.dessertsIndependientes = local.integrantesPartidosDB[0];
                // }
            }   
            else if(tipo == 7){
            await this.$axios.post('/nueva-sesion/genera_rpt_RepresentantesPPSeg', postId, { headers: headers, responseType: 'blob' })
                .then( function(response){
                    //console.table(response.data.integrantesPartidosDB[0]);
                    console.log(response);
                    const blob = new Blob([response.data], {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
                    const url = window.URL.createObjectURL(blob);

                    const a = document.createElement('a');
                    
                    a.href = url;
                    a.download = nombre;
                    a.click();
                    //console.log(a)
                    window.URL.revokeObjectURL(url);
                    //return response.data;
                }).catch( function(error){
                    // console.log(error.response.data.login);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
                // if ( local == false ){
                //     this.abrirPadre();
                // } else {
                // if (local.ok != false ){
                //     //this.dessertsIndependientes = local.integrantesPartidosDB[0];
                // }
            }
        },
        async validaInicioSesion(){
          this.v$.$validate();
          console.table(this.dessertsIntegrantes);
          console.table(this.dessertsPartidos);
          console.table(this.dessertsIndependientes);
          if (!this.v$.form1.$invalid) {
            const { valid } = await (<any>this.$refs.formIntegrantesInicio).validate();
            if ( valid ){
              console.log(this.form1);
              console.log(this.idSesionEdicion);
              console.log(this.dessertsIntegrantes);
              console.log(this.dessertsPartidos);
              console.log(this.dessertsIndependientes);
              const headers = {
                'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              let postId = {
                idSesion: this.idSesionEdicion,
                fecha: this.fechaForms,
                hora: this.form1.hora,
                domicilio: this.form1.domicilio,
                prensa: this.form1.prensa,
                radio: this.form1.radio,
                television: this.form1.television,
                observaciones: this.form1.observaciones,
                dessertsIntegrantes: this.dessertsIntegrantes,
                dessertsPartidos: this.dessertsPartidos,
                dessertsIndependientes: this.dessertsIndependientes,
              };
              this.submitForm = true;
              const Response = await this.$axios.post('/nueva-sesion/realiza-asistencia-inicio', postId, { headers: headers })
              .then( function(response){
                console.log(response.data);
                return response.data;
              }).catch( function(error){
                let login = error.response.data.login;
                if ( login == false ){
                    return false
                } else {
                    return error;
                }
              });
              if ( Response == false ){
                  this.abrirPadre();
              } else {
                this.errors = [];
                if ( Response.ok == true ){
                  this.dialogInicioSesion = false;
                  this.textoTituloSnackbar = 'Correcto';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.form1 = Object.assign({}, this.form1Default);
                  this.snackbar = true;
                  this.llenaTabla();
                  this.submitForm = false;
                } else {
                  this.dialogInicioSesion = false;
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.form1 = Object.assign({}, this.form1Default);
                  this.snackbar = true;
                  this.submitForm = false;
                }
              }
            }
          } else {
          
          }
        },
        async validaFinSesion(){
          this.v$.$validate();
          //router.push('Central') 
          if (!this.v$.form1.$invalid) {
            const { valid } = await (<any>this.$refs.formIntegrantesInicio).validate();
            if ( valid ){
              console.log(this.form1);
              console.log(this.idSesionEdicion);
              console.log(this.dessertsIntegrantes);
              console.log(this.dessertsPartidos);
              console.log(this.dessertsIndependientes);
              const headers = {
                'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              let postId = {
                idSesion: this.idSesionEdicion,
                fecha: this.fechaForms,
                hora: this.form1.hora,
                domicilio: this.form1.domicilio,
                prensa: this.form1.prensa,
                radio: this.form1.radio,
                television: this.form1.television,
                observaciones: this.form1.observaciones,
                dessertsIntegrantes: this.dessertsIntegrantes,
                dessertsPartidos: this.dessertsPartidos,
                dessertsIndependientes: this.dessertsIndependientes,
              };
              this.submitForm = true;
              const Response = await this.$axios.post('/nueva-sesion/cierre-asistencia-inicio', postId, { headers: headers })
              .then( function(response){
                console.log(response.data);
                return response.data;
              }).catch( function(error){
                let login = error.response.data.login;
                if ( login == false ){
                    return false
                } else {
                    return error;
                }
              });
              if ( Response == false ){
                  this.abrirPadre();
              } else {
                this.errors = [];
                if ( Response.ok == true ){
                  this.dialogFinSesion = false;
                  this.textoTituloSnackbar = 'Correcto';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.snackbar = true;
                  this.form1 = Object.assign({}, this.form1Default);
                  this.llenaTabla();
                  this.submitForm = false;
                } else {
                  this.dialogFinSesion = false;
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.form1 = Object.assign({}, this.form1Default);
                  this.snackbar = true;
                  this.submitForm = false;
                }
              }
            }
          }
        },
        validaVotacionPrevio(){
          let contador = <number>0;
          console.table(this.dessertsIntegrantes);
          this.dessertsIntegrantes.forEach((element)=>{
            if (element.votacion != ''){
              contador ++;
            }
          });
          this.contadorVotacionIntegrantes = contador;
          if ( this.contadorAsistenciaIntegrantes !=  contador){
            this.dialogAvisoVota = true;
          } else {
            this.validaVotacion();
          }
        },
        async validaVotacion(){
          console.log('entro');
          this.dialogAvisoVota = false;
          this.v$.$validate();
          const { valid } = await (<any>this.$refs.formVotaciones).validate();
          if ( valid ){
            if (!this.v$.formVotacion.$invalid) {
              console.table(this.dessertsIntegrantes);
              console.log(this.idSesionEdicion);
              console.log(this.idPuntoVotacion);
              const headers = {
                'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              let postId = {
                idSesion: this.idSesionEdicion,
                numPunto: this.idPuntoVotacion,
                observaciones: this.formVotacion.observaciones,
                dessertsIntegrantes: this.dessertsIntegrantes
              };
              this.submitForm = true;
              const Response = await this.$axios.post('/votacion/agregar', postId, { headers: headers })
              .then( function(response){
                console.log(response.data);
                return response.data;
              }).catch( function(error){
                let login = error.response.data.login;
                if ( login == false ){
                    return false
                } else {
                    return error;
                }
              });
              if ( Response == false ){
                  this.abrirPadre();
              } else {
                this.errors = [];
                if ( Response.ok == true ){
                  this.textoTituloSnackbar = 'Correcto';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.snackbar = true;
                  this.idPuntoVotacion = '';
                  this.pasoPuntos = 'window-1';
                  this.formVotacion = Object.assign({}, this.formVotacionDefault);
                  this.consultaPuntosSesion(this.idSesionEdicion);
                  this.v$.$reset();
                  (<any>this.$refs.formVotaciones).reset();
                  this.submitForm = false;
                  this.llenaTabla();
                } else {
                  console.log(Response);
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.response.data.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.snackbar = true;
                  this.idPuntoVotacion = '';
                  this.pasoPuntos = 'window-1';
                  this.formVotacion = Object.assign({}, this.formVotacionDefault);
                  this.v$.$reset();
                  (<any>this.$refs.formVotaciones).reset();
                  this.submitForm = false;
                  this.llenaTabla();
                }
              }
            }
          }
        },
        async validaIntervencion(){
          this.v$.$validate();
          const { valid } = await (<any>this.$refs.formNuevaIntervencion).validate();
          if ( valid ){
            if (!this.v$.form2.$invalid) {
              console.log(this.idSesionEdicion);
              console.log(this.idPuntoVotacion);
              const headers = {
                'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              let postId = {
                idSesion: this.idSesionEdicion,
                numPunto: this.idPuntoVotacion,
                numIntervencion: this.numIntervencion,
                descripcion: this.form2.descripcion,
                replica: this.form2.replica,
                dessertsIntegrantes: this.dessertsIntegrantes,
                dessertsPartidos: this.dessertsPartidos,
                dessertsIndependientes: this.dessertsIndependientes,
              };
              this.submitForm = true;
              const Response = await this.$axios.post('/intervenciones/agregar', postId, { headers: headers })
              .then( function(response){
                console.log(response.data);
                return response.data;
              }).catch( function(error){
                console.log(error);
                let login = error.response.data.login;
                if ( login == false ){
                    return false
                } else {
                    return error;
                }
              });
              if ( Response == false ){
                  this.abrirPadre();
              } else {
                this.errors = [];
                if ( Response.ok == true ){
                  this.textoTituloSnackbar = 'Correcto';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.snackbar = true;
                  this.pasoPuntos = 'window-1';
                  this.form2 = Object.assign({}, this.form2Default);
                  this.v$.$reset();
                  (<any>this.$refs.formNuevaIntervencion).reset();
                  this.submitForm = false;
                } else {
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.response.data.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.pasoPuntos = 'window-1';
                  this.form2 = Object.assign({}, this.form2Default);
                  this.snackbar = true;
                  this.v$.$reset();
                  (<any>this.$refs.formNuevaIntervencion).reset();
                  this.submitForm = false;
                }
              }
            }
          }
        },
        async consultaIntervenciones(item: string){
          this.dessertsIntervencion = [];
          console.log('hola todos');
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: this.idSesionEdicion,
              numPunto: item
          };
          const Response = await this.$axios.post('/intervenciones/consulta-todas/', postId, { headers: headers })
              .then( function(response){
                  console.log(response.data);
                  return response.data;
              }).catch( function(error){
                  console.log('Error: ', error);
                  let login = error.response.data.login;
                  if ( login == false ){
                      return false
                  } else {
                      return error;
                  }
              });
          if ( Response == false ){
              this.abrirPadre();
          } else{
              if ( Response.ok == true ){
                  this.errors = [];
                  console.log(Response.intervencionesDB);
                  this.dessertsIntervencion = Response.intervencionesDB;
              } else {
                  this.errors = [];
                  console.log(Response);
                  this.textoTituloSnackbar = 'Atención';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  //this.snackbar = true;
              }
          }
        },
        async consultaIncidentes(item: string){
          this.dessertsIncidente = [];
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: this.idSesionEdicion,
              numPunto: item
          };
          const Response = await this.$axios.post('/incidentes/consulta-todas/', postId, { headers: headers })
              .then( function(response){
                  console.log(response.data);
                  return response.data;
              }).catch( function(error){
                  console.log('Error: ', error);
                  let login = error.response.data.login;
                  if ( login == false ){
                      return false
                  } else {
                      return error;
                  }
              });
          if ( Response == false ){
              this.abrirPadre();
          } else{
              if ( Response.ok == true ){
                  this.errors = [];
                  console.log(Response.incidentesDB);
                  this.dessertsIncidente = Response.incidentesDB;
              } else {
                  this.errors = [];
                  this.textoTituloSnackbar = 'Atención';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  //this.snackbar = true;
              }
          }
        },
        async validaIncidente(){
          this.v$.$validate();
          const { valid } = await (<any>this.$refs.formNuevoIncidente).validate();
          if ( valid ){
            if (!this.v$.form31.$invalid) {
              console.log(this.idSesionEdicion);
              console.log(this.idPuntoVotacion);
              const headers = {
                'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              let postId = {
                idSesion: this.idSesionEdicion,
                numPunto: this.idPuntoVotacion,
                numIncidente: this.numIncidente,
                descripcion: this.form31.descripcion,
                replica: this.form31.replica,
                dessertsIntegrantes: this.dessertsIntegrantes,
                dessertsPartidos: this.dessertsPartidos,
                dessertsIndependientes: this.dessertsIndependientes,
              };
              this.submitForm = true;
              const Response = await this.$axios.post('/incidentes/agregar', postId, { headers: headers })
              .then( function(response){
                console.log(response);
                return response.data;
              }).catch( function(error){
                let login = error.response.data.login;
                if ( login == false ){
                    return false
                } else {
                    return error;
                }
              });
              if ( Response == false ){
                  this.abrirPadre();
              } else {
                this.errors = [];
                if ( Response.ok == true ){
                  this.textoTituloSnackbar = 'Correcto';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.snackbar = true;
                  this.pasoPuntos = 'window-1';
                  this.form31 = Object.assign({}, this.form31Default);
                  this.v$.$reset();
                  (<any>this.$refs.formNuevoIncidente).reset();
                  this.submitForm = false;
                } else {
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.response.data.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.pasoPuntos = 'window-1';
                  this.form31 = Object.assign({}, this.form31Default);
                  this.snackbar = true;
                  this.v$.$reset();
                  (<any>this.$refs.formNuevoIncidente).reset();
                  this.submitForm = false;
                }
              }
            }
          }
        },
        cerrarDialogInicioSesion (){
          this.v$.$reset();
          this.form1 = Object.assign({}, this.form1Default);
          this.dessertsIntegrantes = [];
          this.dessertsPartidos = [];
          this.dessertsIndependientes = [];
        },
        cerrarDialogFinSesion (){
          this.v$.$reset();
          this.form1 = Object.assign({}, this.form1Default);
          this.dessertsIntegrantes = [];
          this.dessertsPartidos = [];
          this.dessertsIndependientes = [];
        },
        async quitaPuntoSesion(){
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: this.idSesionEdicion,
              numPunto: this.idPuntoVotacion
          };
          this.submitForm = true;
          const local = await this.$axios.post('/inhabilitados/agrega', postId, { headers: headers })
          .then( function(response){
              console.table(response.data);
              return response.data;
          }).catch( function(error){
            console.log(error);
              let login = error.response.data.login;
              if ( login == false ){
                  return false
              } else {
                  return error;
              }
          });
          if ( local == false ){
              this.abrirPadre();
          } else {
            if ( local.ok == true ){
              this.dialogQuitaPunto = false;
              this.textoTituloSnackbar = 'Correcto';
              this.textoSnackbar = local.msg;
              this.colorSnackbar = this.color;
              this.snackbar = true;
              this.consultaPuntosSesion(this.idSesionEdicion);
              this.submitForm = false;
            } else {
              this.dialogQuitaPunto = false;
              this.textoTituloSnackbar = 'Error';
              this.textoSnackbar = local.msg;
              this.colorSnackbar = 'red-darken-2';
              this.snackbar = true;
              this.submitForm = false;
            }
          }
        },
        async habilitaPuntoSesion(){
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: this.idSesionEdicion,
              numPunto: this.idPuntoVotacion
          };
          this.submitForm = true;
          const local = await this.$axios.post('/inhabilitados/borra', postId, { headers: headers })
          .then( function(response){
              console.table(response.data);
              return response.data;
          }).catch( function(error){
            console.log(error);
              let login = error.response.data.login;
              if ( login == false ){
                  return false
              } else {
                  return error;
              }
          });
          if ( local == false ){
              this.abrirPadre();
          } else {
            if ( local.ok == true ){
              this.dialogAgregaQPunto = false;
              this.textoTituloSnackbar = 'Correcto';
              this.textoSnackbar = local.msg;
              this.colorSnackbar = this.color;
              this.snackbar = true;
              this.consultaPuntosSesion(this.idSesionEdicion);
              this.submitForm = false;
            } else {
              this.dialogAgregaQPunto = false;
              this.textoTituloSnackbar = 'Error';
              this.textoSnackbar = local.msg;
              this.colorSnackbar = 'red-darken-2';
              this.snackbar = true;
              this.submitForm = false;
            }
          }
        },
        cierraDialogOrdenDia(){
          this.v$.$reset();
          this.desserts3 = [];
          this.idSesionEdicion = 0;
          this.pasoPuntos = 'window-1';
          this.form2 = Object.assign({}, this.form2Default);
          this.form31 = Object.assign({}, this.form31Default);
          this.numIncidente = 1;
          this.numIntervencion = 1;
        },
        async consultaVotacionesIntegrantges(item: number){
          console.log(item);
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: item
          };
          const local = await this.$axios.post('/asistencia/consulta-asistencia-consejo', postId, { headers: headers })
          .then( function(response){
              console.table(response.data.AsistenciaDB[0]);
              return response.data.AsistenciaDB[0];
          }).catch( function(error){
              let login = error.response.data.login;
              if ( login == false ){
                  return false
              } else {
                  return error;
              }
          });
          if ( local == false ){
              this.abrirPadre();
          } else {
            this.dessertsIntegrantes = local;
            //console.table(this.dessertsIntegrantes);
          }
        },
        //Array<any>
        abreVotaciones(item: string){
          console.log(item);
          this.pasoPuntos = 'window-2';
          this.consultaVotacionesIntegrantges(this.idSesionEdicion);
          this.idPuntoVotacion = item; 
        },
        abreNuevoIntervencion(item: string){
          this.idPuntoVotacion = item; 
          this.consultaIntegrantesInicioSesion(this.idSesionEdicion);
          this.consultaIntegrantesPartidosInicioSesion(this.idSesionEdicion);
          this.consultaIndepPartidosInicioSesion(this.idSesionEdicion);
          this.pasoPuntos = 'window-3';
        },
        abreEditaIntervencion(item: string){
          console.log(item);
          this.idPuntoVotacion = item; 
          this.dessertsIntervencion = [];
          this.consultaIntervenciones(item);
          this.pasoPuntos = 'window-4';
        },
        abreNuevoIncidente(item: string){
          this.idPuntoVotacion = item; 
          this.consultaIntegrantesInicioSesion(this.idSesionEdicion);
          this.consultaIntegrantesPartidosInicioSesion(this.idSesionEdicion);
          this.consultaIndepPartidosInicioSesion(this.idSesionEdicion);
          this.pasoPuntos = 'window-5';
        },
        abreEditaIncidente(item: string){
          this.idPuntoVotacion = item; 
          this.consultaIncidentes(item);
          this.pasoPuntos = 'window-6';
        },
        abreQuitaPunto(item: any){
          console.log(item);
          this.idSesionEdicion = parseInt(item.idSesion);
          this.idPuntoVotacion = item.numPunto;
          this.dialogQuitaPunto = true;
        },
        abreAgregaPunto(item: any){
          console.log(item);
          this.idSesionEdicion = parseInt(item.idSesion);
          this.idPuntoVotacion = item.numPunto;
          this.dialogAgregaQPunto = true;
        },
        doc_sesion(item: number, file: any){
          console.log(item);
          console.log(file);
          this.formDocSesion.idSesion = item;
          this.formDocSesion.documentoSesion = file;
          this.dialogDocumentos = true;
        },
        acta(item: number, file: any){
          console.log(file);
          this.formActaSesion.idSesion = item;
          this.formActaSesion.actaSesion = file;
          this.dialogActa = true;
        },
        minuta(item: number, file: any){
          console.log(file);
          this.formMinutaSesion.idSesion = item;
          this.formMinutaSesion.minutaSesion = file;
          this.dialogMinutas = true;
        },
        decrement(){
          if( this.numIntervencion > 1 ){
              this.numIntervencion --;
          }
        },
        increment(){
          if( this.numIntervencion < 6 ){
            this.numIntervencion ++;
          }
        },
        decrementEdited(){
            if( this.form2Edited.numIntervencion > 1 ){
                this.form2Edited.numIntervencion --;
            }
        },
        incrementEdited(){
            this.form2Edited.numIntervencion ++;
        },
        decrement3(){
          if( this.numIncidente > 1 ){
              this.numIncidente --;
          }
        },
        increment3(){
          if( this.numIncidente < 3 ){
            this.numIncidente ++;
          }
        },
        decrement3Edited(){
            if( this.form31Edited.numIncidente > 1 ){
                this.form31Edited.numIncidente --;
            }
        },
        increment3Edited(){
            this.form31Edited.numIncidente ++;
        },
        async consultaIntervencion(item: any){
          //this.dessertsIntervencion = [];
          console.log('hola todos');
          console.log(item);
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: this.idSesionEdicion,
              idIntervencion: item.idIntervencion,
              numPunto: item.numPunto
          };
          const Response = await this.$axios.post('/intervenciones/consulta-una', postId, { headers: headers })
              .then( function(response){
                  console.log(response.data);
                  return response.data;
              }).catch( function(error){
                  console.log('Error: ', error);
                  let login = error.response.data.login;
                  if ( login == false ){
                      return false
                  } else {
                      return error;
                  }
              });
          if ( Response == false ){
              this.abrirPadre();
          } else{
              if ( Response.ok == true ){
                  this.errors = [];
                  console.log(Response);
                  this.form2.descripcion = Response.intervencionDB.descripcionEvento;
                  this.form2.replica = Response.intervencionDB.replica;
                  this.numIntervencion = Response.intervencionDB.idIntervencion;
                  this.form2.numPunto = Response.intervencionDB.numPunto;
                  this.dessertsIntegrantes = Response.involucradosConsejo;
                  this.dessertsPartidos = Response.involucradosPartidos;
                  this.dessertsIndependientes = Response.involucradosIndependientes;
              } else {
                  this.errors = [];
                  console.log(Response);
                  this.textoTituloSnackbar = 'Atención';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  //this.snackbar = true;
              }
          }
          this.idIntervencionesEdited = item;
          this.pasoPuntos = 'window-31';
        },
        async validaIntervencionEdicion(){
          this.v$.$validate();
          console.table(this.dessertsIntegrantes);
          console.table(this.dessertsPartidos);
          console.table(this.dessertsIndependientes);
          if (!this.v$.form2.$invalid) {
            const { valid } = await (<any>this.$refs.formEditaIntervencion).validate();
            if ( valid ){
              console.log(this.form2);
              console.log(this.idSesionEdicion);
              console.log(this.dessertsIntegrantes);
              console.log(this.dessertsPartidos);
              console.log(this.dessertsIndependientes);
              const headers = {
                'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              let postId = {
                idSesion: this.idSesionEdicion,
                descripcion: this.form2.descripcion,
                idIntervencion: this.numIntervencion,
                numPunto: this.form2.numPunto,
                replica: this.form2.replica,
                dessertsIntegrantes: this.dessertsIntegrantes,
                dessertsPartidos: this.dessertsPartidos,
                dessertsIndependientes: this.dessertsIndependientes,
              };
              this.submitForm = true;
              const Response = await this.$axios.post('/intervenciones/actualiza-intervencion', postId, { headers: headers })
              .then( function(response){
                console.log(response.data);
                return response.data;
              }).catch( function(error){
                let login = error.response.data.login;
                if ( login == false ){
                    return false
                } else {
                    return error;
                }
              });
              if ( Response == false ){
                  this.abrirPadre();
              } else {
                this.errors = [];
                if ( Response.ok == true ){
                  this.textoTituloSnackbar = 'Correcto';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.form2 = Object.assign({}, this.form2Default);
                  this.snackbar = true;
                  this.pasoPuntos = 'window-1';
                  this.submitForm = false;
                } else {
                  this.dialogInicioSesion = false;
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.form2 = Object.assign({}, this.form2Default);
                  this.snackbar = true;
                  this.submitForm = false;
                }
              }
            }
          } else {
          
          }
        },
        async consultaIncidente(item: any){
          console.log(item);
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: this.idSesionEdicion,
              idIncidente: item.idIncidente,
              numPunto: item.numPunto
          };
          const Response = await this.$axios.post('/incidentes/consulta-una', postId, { headers: headers })
              .then( function(response){
                  console.log(response.data);
                  return response.data;
              }).catch( function(error){
                  console.log('Error: ', error);
                  let login = error.response.data.login;
                  if ( login == false ){
                      return false
                  } else {
                      return error;
                  }
              });
          if ( Response == false ){
              this.abrirPadre();
          } else{
              if ( Response.ok == true ){
                  this.errors = [];
                  console.log(Response);
                  this.form31.descripcion = Response.incienteDB.descripcionEvento;
                  this.form31.replica = Response.incienteDB.replica;
                  this.numIncidente = Response.incienteDB.idIncidente;
                  this.form31.numPunto = Response.incienteDB.numPunto;
                  this.dessertsIntegrantes = Response.involucradosConsejo;
                  this.dessertsPartidos = Response.involucradosPartidos;
                  this.dessertsIndependientes = Response.involucradosIndependientes;
              } else {
                  this.errors = [];
                  console.log(Response);
                  this.textoTituloSnackbar = 'Atención';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  //this.snackbar = true;
              }
          }
          this.idIncidenteEdited = item;
          this.pasoPuntos = 'window-61';
        },
        async validaIncidenteEdicion(){
          this.v$.$validate();
          console.table(this.dessertsIntegrantes);
          console.table(this.dessertsPartidos);
          console.table(this.dessertsIndependientes);
          if (!this.v$.form31.$invalid) {
            const { valid } = await (<any>this.$refs.formNuevoIncidente).validate();
            if ( valid ){
              console.log(this.form31);
              console.log(this.idSesionEdicion);
              console.log(this.dessertsIntegrantes);
              console.log(this.dessertsPartidos);
              console.log(this.dessertsIndependientes);
              const headers = {
                'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              let postId = {
                idSesion: this.idSesionEdicion,
                descripcion: this.form31.descripcion,
                idIncidente: this.numIncidente,
                numPunto: this.form31.numPunto,
                replica: this.form31.replica,
                dessertsIntegrantes: this.dessertsIntegrantes,
                dessertsPartidos: this.dessertsPartidos,
                dessertsIndependientes: this.dessertsIndependientes,
              };
              this.submitForm = true;
              const Response = await this.$axios.post('/incidentes/actualiza-incidente', postId, { headers: headers })
              .then( function(response){
                console.log(response.data);
                return response.data;
              }).catch( function(error){
                let login = error.response.data.login;
                if ( login == false ){
                    return false
                } else {
                    return error;
                }
              });
              if ( Response == false ){
                  this.abrirPadre();
              } else {
                this.errors = [];
                if ( Response.ok == true ){
                  this.textoTituloSnackbar = 'Correcto';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.form31 = Object.assign({}, this.form31Default);
                  this.snackbar = true;
                  this.pasoPuntos = 'window-1';
                  this.submitForm = false;
                } else {
                  this.dialogInicioSesion = false;
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.form31 = Object.assign({}, this.form31Default);
                  this.snackbar = true;
                  this.submitForm = false;
                }
              }
            }
          } else {
          
          }
        },
        eliminaIntervencion(item: number){
          this.idIntervencionesEdited = item;
          this.dialogQuitaIntervencion = true;
        },
        cierraDialogQuitaIntervencion(){
          this.idIntervencionesEdited = 0;
        },
        eliminaIncidente(item: number){
          this.idIncidenteEdited = item;
          this.dialogQuitaIncidente = true;
        },
        cierraDialgoQuitaIncidente(){
          this.idIncidenteEdited = 0;
        },
        async borrarIntervencion(){
          console.log(this.idPuntoVotacion);
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: this.idSesionEdicion,
              idIntervencion: this.idIntervencionesEdited,
              numPunto: this.idPuntoVotacion
          };
          this.submitForm = true;
          const local = await this.$axios.post('/intervenciones/eliminar', postId, { headers: headers })
          .then( function(response){
            console.table(response);
            return response.data;
          }).catch( function(error){
            console.log(error);
            let login = error.response.data.login;
            if ( login == false ){
                return false
            } else {
                return error;
            }
          });
          if ( local == false ){
              this.abrirPadre();
          } else {
            this.errors = [];
            if ( local.ok == true ){
              this.textoTituloSnackbar = 'Correcto';
              this.textoSnackbar = local.msg;
              this.colorSnackbar = this.color;
              this.snackbar = true;
              this.dialogQuitaIntervencion = false;
              this.consultaIntervenciones(this.idPuntoVotacion);
              this.submitForm = false;
            } else {
              this.textoTituloSnackbar = 'Error';
              this.textoSnackbar = local.response.data.msg;
              this.colorSnackbar = 'red-darken-2';
              this.snackbar = true;
              this.dialogQuitaIntervencion = false;
              this.submitForm = false;
            }
            //console.table(this.dessertsIntegrantes);
          }
        },
        async borrarIncidente(){
          console.log(this.idPuntoVotacion);
          const headers = {
              'Content-Type': 'application/json',
              'x-token-admin':localStorage.jwt
          }
          let postId = {
              idSesion: this.idSesionEdicion,
              idIncidente: this.idIncidenteEdited,
              numPunto: this.idPuntoVotacion
          };
          this.submitForm = true;
          const local = await this.$axios.post('/incidentes/eliminar', postId, { headers: headers })
          .then( function(response){
            console.table(response);
            return response.data;
          }).catch( function(error){
            console.log(error);
            let login = error.response.data.login;
            if ( login == false ){
                return false
            } else {
                return error;
            }
          });
          if ( local == false ){
              this.abrirPadre();
          } else {
            this.errors = [];
            if ( local.ok == true ){
              this.textoTituloSnackbar = 'Correcto';
              this.textoSnackbar = local.msg;
              this.colorSnackbar = this.color;
              this.snackbar = true;
              this.dialogQuitaIncidente = false;
              this.consultaIncidentes(this.idPuntoVotacion);
              this.submitForm = false;
            } else {
              this.textoTituloSnackbar = 'Error';
              this.textoSnackbar = local.response.data.msg;
              this.colorSnackbar = 'red-darken-2';
              this.snackbar = true;
              this.dialogQuitaIncidente = false;
              this.submitForm = false;
            }
            //console.table(this.dessertsIntegrantes);
          }
        },
        cierraDialogDocumentoSesion(){
          this.v$.$reset();
          this.$nextTick(() => {
            this.formDocSesion = Object.assign({}, this.formDocSesionDefault);
          });
        },
        async validaDocumentoSesion(){
          this.v$.$validate();
          if (!this.v$.formDocSesion.$invalid) {
            let formData = new FormData();
            formData.append('idSesion', <any>this.formDocSesion.idSesion);
            formData.append('documento', <any>this.formDocSesion.file[0]);
            if ( this.formDocSesion.file[0].type == 'application/pdf' || this.formDocSesion.file[0].type == 'application/zip' ||  this.formDocSesion.file[0].type == 'application/x-zip-compressed'){
              const headers = {
                'Accept': 'application/json',
                'Content-Type': 'multipart/form-data;',
                //'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              this.submitForm = true;
              const Response = await this.$axios.post('/nueva-sesion/agrega-documento-sesion/'+ this.formDocSesion.idSesion + '/' + localStorage.idDistrito, formData, { headers: headers })
                .then( function(response){
                    console.log(response.data);
                    return response.data;
                }).catch( function(error){
                    console.log('Error: ', error);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
              if ( Response == false ){
                  this.abrirPadre();
              } else{
                if ( Response.ok == true ){
                  this.errors = [];
                  this.textoTituloSnackbar = 'Guardado';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.snackbar = true;
                  this.v$.$reset();
                  this.$nextTick(() => {
                    this.formDocSesion = Object.assign({}, this.formDocSesionDefault);
                  });
                  this.dialogDocumentos = false;
                  this.llenaTabla();
                  this.submitForm = false;
                } else {
                  this.errors = [];
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.msg;
                  this.textoSnackbar = Response.response.data.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.snackbar = true;
                  this.submitForm = false;
                }
              }
            } else {
              this.errors = [];
              this.errors.push('El formato del archivo no es PDF o ZIP');
              this.snackbar = true;
              this.colorSnackbar = 'red-darken-2';  
            }
          }
        },
        cierraDialogActaSesion(){
          this.v$.$reset();
          this.$nextTick(() => {
            this.formActaSesion = Object.assign({}, this.formActaSesionDefault);
          });
        },
        async validaActaSesion(){
          this.v$.$validate();
          if (!this.v$.formActaSesion.$invalid) {
            let formData = new FormData();this.idSesionEdicion 
            formData.append('idSesion', <any>this.formActaSesion.idSesion);
            formData.append('documento', <any>this.formActaSesion.file[0]);
            if ( this.formActaSesion.file[0].type == 'application/pdf' || this.formActaSesion.file[0].type == 'application/zip' ||  this.formActaSesion.file[0].type == 'application/x-zip-compressed'){
              const headers = {
                'Accept': 'application/json',
                'Content-Type': 'multipart/form-data;',
                //'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              this.submitForm = true;
              const Response = await this.$axios.post('/nueva-sesion/agrega-acta-sesion/'+ this.formActaSesion.idSesion + '/' + localStorage.idDistrito, formData, { headers: headers })
                .then( function(response){
                    console.log(response.data);
                    return response.data;
                }).catch( function(error){
                    console.log('Error: ', error);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
              if ( Response == false ){
                  this.abrirPadre();
              } else{
                if ( Response.ok == true ){
                  this.errors = [];
                  this.textoTituloSnackbar = 'Guardado';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.snackbar = true;
                  this.v$.$reset();
                  this.$nextTick(() => {
                    this.formActaSesion = Object.assign({}, this.formActaSesionDefault);
                  });
                  this.dialogActa = false;
                  this.llenaTabla();
                  this.submitForm = false;
                } else {
                  this.errors = [];
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.msg;
                  this.textoSnackbar = Response.response.data.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.snackbar = true;
                  this.submitForm = false;
                }
              }
            } else {
              this.errors = [];
              this.errors.push('El formato del archivo no es PDF o ZIP');
              this.snackbar = true;
              this.colorSnackbar = 'red-darken-2';  
            }
          }
        },
        cierraDialogMinutaSesion(){
          this.v$.$reset();
          this.$nextTick(() => {
            this.formMinutaSesion = Object.assign({}, this.formMinutaSesionDefault);
          });
        },
        async validaMinutaSesion(){
          this.v$.$validate();
          if (!this.v$.formMinutaSesion.$invalid) {
            let formData = new FormData();
            formData.append('idSesion', <any>this.formMinutaSesion.idSesion);
            formData.append('documento', <any>this.formMinutaSesion.file[0]);
            if ( this.formMinutaSesion.file[0].type == 'application/pdf' || this.formMinutaSesion.file[0].type == 'application/zip' ||  this.formMinutaSesion.file[0].type == 'application/x-zip-compressed'){
              const headers = {
                'Accept': 'application/json',
                'Content-Type': 'multipart/form-data;',
                //'Content-Type': 'application/json',
                'x-token-admin':localStorage.jwt
              }
              this.submitForm = true;
              const Response = await this.$axios.post('/nueva-sesion/agrega-minuta-sesion/'+ this.formMinutaSesion.idSesion + '/' + localStorage.idDistrito, formData, { headers: headers })
                .then( function(response){
                    console.log(response.data);
                    return response.data;
                }).catch( function(error){
                    console.log('Error: ', error);
                    let login = error.response.data.login;
                    if ( login == false ){
                        return false
                    } else {
                        return error;
                    }
                });
              if ( Response == false ){
                  this.abrirPadre();
              } else{
                if ( Response.ok == true ){
                  this.errors = [];
                  this.textoTituloSnackbar = 'Guardado';
                  this.textoSnackbar = Response.msg;
                  this.colorSnackbar = this.color;
                  this.snackbar = true;
                  this.v$.$reset();
                  this.$nextTick(() => {
                    this.formMinutaSesion = Object.assign({}, this.formMinutaSesionDefault);
                  });
                  this.dialogMinutas = false;
                  this.llenaTabla();
                  this.submitForm = false;
                } else {
                  this.errors = [];
                  this.textoTituloSnackbar = 'Error';
                  this.textoSnackbar = Response.msg;
                  this.textoSnackbar = Response.response.data.msg;
                  this.colorSnackbar = 'red-darken-2';
                  this.snackbar = true;
                  this.submitForm = false;
                }
              }
            } else {
              this.errors = [];
              this.errors.push('El formato del archivo no es PDF o ZIP');
              this.snackbar = true;
              this.colorSnackbar = 'red-darken-2';  
            }
          }
        },
        cierraDialogInicio(){
          this.$nextTick(() => {
            this.form1 = Object.assign({}, this.form1Default);
          });

        },
        cierraDialogFin(){
          this.$nextTick(() => {
            this.form1 = Object.assign({}, this.form1Default);
          });
        }
      },      
      watch: {
        color(newColor){
          localStorage.color = newColor;
        },
        dialogIntegrantesConsejo(val){
				  val || this.cierraDialogConsejo();
        },
        dialogInicioSesion(val){
				  val || this.cerrarDialogInicioSesion();
        },
        dialogFinSesion(val){
				  val || this.cerrarDialogFinSesion();
        },
        dialogOrdenDDia(val){
          val || this.cierraDialogOrdenDia();
        },
        dialogQuitaIntervencion(val){
          val || this.cierraDialogQuitaIntervencion();
        },
        dialogQuitaIncidente(val){
          val || this.cierraDialgoQuitaIncidente()
        },
        dialogDocumentos(val){
          val || this.cierraDialogDocumentoSesion()
        },
        dialogActa(val){
          val || this.cierraDialogActaSesion()
        },
        dialogMinutas(val){
          val || this.cierraDialogMinutaSesion()
        },
        dialogEstado_sesion(val){
          val || this.cierraDialogEstado_sesion()
        }
      },
      mounted(){
        const colorStore = useSisecedStore();
        console.log(colorStore.color);
        this.color = colorStore.color;
        if (localStorage.color) {
            this.color = localStorage.color;
        }
        this.listadoSesiones();
      },
        
    }
</script>