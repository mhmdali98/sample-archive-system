<template>
  <v-container id="dashboard" fluid tag="section">
    <v-text-field
      class="mt-4"
      label="اكتب للبحث"
      outlined
      append-icon="mdi-magnify"
      v-model="search"
    ></v-text-field>
    <v-data-table
      :search="search"
      :headers="headers"
      :items="desserts"
      sort-by="phone"
      class="elevation-1"
      :loading="loading"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>قائمة المستخدمين</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">اضافة مستخدم</v-btn>
            </template>
            <v-card>
              <v-toolbar dark color="primary lighten-1 mb-5">
                <v-toolbar-title>
                  <h3>{{ formTitle }}</h3>
                </v-toolbar-title>
                <v-spacer />
                <v-btn @click="dialog = !dialog" icon>
                  <v-icon>mdi-close</v-icon>
                </v-btn>
              </v-toolbar>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col class="py-0" cols="12" sm="12" md="12">
                      <v-text-field
                        v-model="editedItem.userName"
                        label="اسم المستخدم"
                        outlined
                        :rules="[rules.required]"
                      ></v-text-field>
                    </v-col>
                    <v-col class="py-0" cols="12" sm="12" md="12" v-if="editedIndex === -1">
                      <v-text-field
                        v-model="editedItem.password"
                        label="كلمة المرور"
                        outlined
                        :rules="[rules.required, rules.min]"
                        :type="show1 ? 'text' : 'password'"
                        :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
                        @click:append="show1 = !show1"
                      ></v-text-field>
                    </v-col>
                    <v-col class="py-0" cols="12" sm="12" md="12">
                      <v-text-field
                        v-model="editedItem.fullName"
                        label="الاسم الكامل"
                        outlined
                        :rules="[rules.required]"
                      ></v-text-field>
                    </v-col>
                    <v-col class="py-0" cols="12" sm="12" md="12">
                      <v-select
                        v-model="editedItem.ognaizationId"
                        :items="ognaizationList"
                        label="اختر القسم"
                        outlined
                        item-text="name"
                        item-value="id"
                        :rules="[rules.required]"
                      ></v-select>
                    </v-col>
                    <v-col class="py-0" cols="12" sm="12" md="12">
                      <v-select
                        v-model="editedItem.roleId"
                        :items="perAll"
                        label="اختر الصلاحية"
                        outlined
                        item-text="name"
                        item-value="id"
                        :rules="[rules.required]"
                      ></v-select>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red darken-1" text @click="close">الغاء</v-btn>
                <v-btn :loading="loadSave" color="blue darken-1" text @click="save">حفظ</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.isDeleted="{ item }">
        <v-chip v-if="!item.isDeleted" color="green" dark>مفعل</v-chip>
        <v-chip v-if="item.isDeleted">غير مفعل</v-chip>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-icon
              class="ml-5"
              @click="editItem(item)"
              v-if="!item.isDeleted"
              v-bind="attrs"
              v-on="on"
              >mdi-pencil</v-icon
            >
          </template>
          <span>تعديل</span>
        </v-tooltip>
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-icon @click="deleteItem(item)" v-if="!item.isDeleted" v-bind="attrs" v-on="on"
              >mdi-delete</v-icon
            >
          </template>
          <span>الغاء التفعيل</span>
        </v-tooltip>
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-icon @click="deleteItem(item)" v-if="item.isDeleted" v-bind="attrs" v-on="on"
              >mdi-reload</v-icon
            >
          </template>
          <span>تفعيل</span>
        </v-tooltip>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">اعادة تحميل</v-btn>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
import Axios from "axios";
import Swal from "sweetalert2";

export default {
  data: () => ({
    show1: false,
    rules: {
      required: value => !!value || "مطلوب",
      min: v => v.length >= 6 || "كلمة المرور يجب ان تتكون من 6 عناصر او اكثر",
      minPhon: v => v.length == 13 || "رقم الهاتف يجب ان يتكون من 11 رقم"
    },
    search: "",
    loading: false,
    loadSave: false,
    dialog: false,
    headers: [
      {
        text: "اسم المستخدم",
        align: "start",
        value: "fullName"
      },
      { text: " اسم الحساب", value: "userName" },
      { text: " القسم ", value: "ognaizationName" },
      { text: " الصلاحية", value: "roleName" },
      { text: "الحالة", value: "isDeleted" },
      { text: "العمليات", value: "actions", sortable: false }
    ],
    desserts: [],
    ognaizationList: [],
    perAll: [
      { id: 1, name: "ادارة عامة" },
      { id: 2, name: "مستخدم" }
    ],
    editedIndex: -1,
    editedItem: {
      userName: "",
      password: "",
      fullName: "",
      ognaizationId: "",
      id: "",
      roleId: ""
    },
    defaultItem: {
      userName: "",
      password: "",
      fullName: "",
      ognaizationId: "",
      roleId: ""
    }
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "اضافة مستخدم" : "تعديل المستخدم";
    }
  },

  watch: {
    dialog(val) {
      val || this.close();
    }
  },

  created() {
    this.initialize();
  },

  methods: {
    showUsers() {
      this.loading = true;
      Axios.get("user")
        .then(res => {
          this.loading = false;
          this.desserts = res.data;
          // console.log(res.data);
        })
        .catch(() => {
          this.loading = false;
        });
    },
    getOrg() {
      Axios.get("Ognaization")
        .then(res => {
          this.ognaizationList = res.data;
          // console.log(res.data);
        })
        .catch(() => {});
    },
    initialize() {
      this.showUsers();
      this.getOrg();
    },

    editItem(item) {
      this.editedIndex = this.desserts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      // console.log(this.editedItem);
      this.editedItem.phone = this.editedItem.phone;

      this.dialog = true;
    },

    deleteItem(item) {
      Swal.fire({
        title: "هل انت متاكد من العملية  ؟ ",
        text: "",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "نعم",
        cancelButtonText: "لا"
      }).then(result => {
        if (result.value) {
          Axios.delete("user?Id=" + item.id)
            .then(() => {
              Swal.fire("تم ", "تمت العملية  بنجاح", "success");
              this.showUsers();
            })
            .catch(() => {
              Swal.fire("لم يتم ", "لم تمت العملية  بنجاح", "error");
            });
        }
      });
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      this.loadSave = true;
      if (this.editedIndex > -1) {
        const editData = {
          name: this.editedItem.username,
          phoneNumber: this.editedItem.phone.replace(/ /g, ""),
          role: this.editedItem.perID
        };
        Axios.put("account/edit?id=" + this.editedItem.id, editData, {
          headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
            Authorization: "Bearer " + this.$store.state.idToken
          }
        })
          .then(() => {
            this.loadSave = false;
            this.showUsers();
            this.close();
            Swal.fire({
              title: "تم تعديل المستخدم",
              text: "",
              icon: "success",
              confirmButtonText: "اغلاق"
            });
          })
          .catch(() => {
            this.loadSave = false;

            Swal.fire({
              title: "تاكد من ملى المعلومات",
              text: "",
              icon: "error",
              confirmButtonText: "اغلاق"
            });
          });
      } else if (
        this.editedItem.userName &&
        this.editedItem.password &&
        this.editedItem.fullName &&
        this.editedItem.roleId &&
        this.editedItem.ognaizationId
      ) {
        Axios.post("user", this.editedItem)
          .then(() => {
            this.loadSave = false;
            this.showUsers();
            this.close();
            Swal.fire({
              title: "تم اضافة المستخدم",
              text: "",
              icon: "success",
              confirmButtonText: "اغلاق"
            });
          })
          .catch(err => {
            // console.log(err.response.data);
            this.loadSave = false;
            if (err.response.data.code == 400.2) {
              Swal.fire({
                title: "اسم المستخدم موجود",
                text: "",
                icon: "error",
                confirmButtonText: "اغلاق"
              });
            } else {
              Swal.fire({
                title: "تاكد من ملى المعلومات",
                text: "",
                icon: "error",
                confirmButtonText: "اغلاق"
              });
            }
          });
      } else {
        this.loadSave = false;

        Swal.fire({
          title: "تاكد من ملى المعلومات",
          text: "",
          icon: "error",
          confirmButtonText: "اغلاق"
        });
      }
    }
  }
};
</script>

<style></style>
