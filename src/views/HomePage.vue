<template>
  <ion-page>
    <ion-content>
      <div class="container">

        <div class="header">
          <h1>Inventory System</h1>
          <p>Manage your inventory items</p>
        </div>

        <ion-card class="form-card">
          <ion-card-header>
            <ion-card-title>
              {{ editingId ? 'Update Item' : 'Add New Item' }}
            </ion-card-title>
          </ion-card-header>

          <ion-card-content>

            <ion-item>
              <ion-input
                v-model="item.itemName"
                label="Item Name"
                label-placement="floating"
                placeholder="Enter item name"
              />
            </ion-item>

            <ion-item>
              <ion-input
                v-model="item.quantity"
                type="number"
                label="Quantity"
                label-placement="floating"
                placeholder="Enter quantity"
              />
            </ion-item>

            <ion-item>
              <ion-select
                v-model="item.category"
                label="Category"
                label-placement="floating"
                placeholder="Select category"
              >
                <ion-select-option value="Food">
                  Food
                </ion-select-option>

                <ion-select-option value="Beverage">
                  Beverage
                </ion-select-option>

                <ion-select-option value="Electronics">
                  Electronics
                </ion-select-option>

                <ion-select-option value="School Supplies">
                  School Supplies
                </ion-select-option>

                <ion-select-option value="Other">
                  Other
                </ion-select-option>
              </ion-select>
            </ion-item>

            <ion-item>
              <ion-input
                v-model="item.price"
                type="number"
                label="Price"
                label-placement="floating"
                placeholder="Enter price"
              />
            </ion-item>

            <ion-item>
              <ion-input
                :value="item.stockStatus"
                label="Stock Status"
                label-placement="floating"
                placeholder="Automatically calculated"
                readonly
              />
            </ion-item>

            <div class="button-container">
              <ion-button
                expand="block"
                @click="saveItem"
              >
                {{ editingId ? 'Update Item' : 'Save Item' }}
              </ion-button>

              <ion-button
                v-if="editingId"
                expand="block"
                fill="outline"
                color="medium"
                @click="cancelEdit"
              >
                Cancel
              </ion-button>
            </div>

          </ion-card-content>
        </ion-card>

        <div class="inventory-header">
          <div>
            <h2>Inventory</h2>
            <p>{{ itemList.length }} item(s)</p>
          </div>
        </div>

        <ion-card v-if="itemList.length === 0">
          <ion-card-content class="empty">
            <h3>No Items Yet</h3>
            <p>
              Add your first inventory item above.
            </p>
          </ion-card-content>
        </ion-card>

        <div
          v-for="i in itemList"
          :key="i.id"
          class="inventory-card"
        >

          <div class="item-header">
            <div>
              <h3>
                {{ i.itemName }}
              </h3>

              <span class="category">
                {{ i.category }}
              </span>
            </div>

            <span
              class="status"
              :class="{
                'status-in': i.stockStatus === 'In Stock',
                'status-low': i.stockStatus === 'Low Stock',
                'status-out': i.stockStatus === 'Out of Stock'
              }"
            >
              {{ i.stockStatus }}
            </span>
          </div>

          <div class="details">

            <div class="detail">
              <span class="label">
                Quantity
              </span>

              <strong>
                {{ i.quantity }}
              </strong>
            </div>

            <div class="detail">
              <span class="label">
                Price
              </span>

              <strong>
                ₱{{ Number(i.price).toFixed(2) }}
              </strong>
            </div>

          </div>

          <div class="actions">

            <ion-button
              fill="outline"
              size="small"
              @click="editItem(i)"
            >
              Edit
            </ion-button>

            <ion-button
              fill="outline"
              color="danger"
              size="small"
              @click="deleteItem(i.id)"
            >
              Delete
            </ion-button>

          </div>

        </div>

      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">

import { ref, onMounted } from 'vue';

import {
  IonPage,
  IonContent,
  IonItem,
  IonInput,
  IonSelect,
  IonSelectOption,
  IonButton,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent
} from '@ionic/vue';

import { db } from '@/firebase';

import {
  ref as dbRef,
  push,
  onValue,
  update,
  remove
} from 'firebase/database';

const item = ref({
  itemName: '',
  quantity: '',
  category: '',
  price: '',
  stockStatus: ''
});

const itemList = ref<any[]>([]);

const editingId = ref<string | null>(null);

onMounted(() => {

  const dbConnection = dbRef(
    db,
    'inventory'
  );

  onValue(
    dbConnection,
    (snapshot) => {

      const rawData = snapshot.val();

      if (rawData) {

        itemList.value =
          Object.keys(rawData)
            .map(key => ({
              id: key,
              ...rawData[key]
            }))
            .reverse();

      } else {

        itemList.value = [];

      }

    }
  );

});

const saveItem = async () => {

  if (
    !item.value.itemName ||
    item.value.quantity === '' ||
    !item.value.category ||
    item.value.price === ''
  ) {

    alert(
      'Please complete all fields.'
    );

    return;

  }

  const quantity =
    Number(item.value.quantity);

  const price =
    Number(item.value.price);

  let stockStatus = '';

  if (quantity === 0) {

    stockStatus =
      'Out of Stock';

  } else if (quantity <= 10) {

    stockStatus =
      'Low Stock';

  } else {

    stockStatus =
      'In Stock';

  }

  if (editingId.value) {

    const itemReference = dbRef(
      db,
      `inventory/${editingId.value}`
    );

    await update(
      itemReference,
      {
        itemName:
          item.value.itemName,

        quantity:
          quantity,

        category:
          item.value.category,

        price:
          price,

        stockStatus:
          stockStatus
      }
    );

    alert(
      'Item updated successfully!'
    );

  } else {

    const dbConnection =
      dbRef(db, 'inventory');

    await push(
      dbConnection,
      {
        itemName:
          item.value.itemName,

        quantity:
          quantity,

        category:
          item.value.category,

        price:
          price,

        stockStatus:
          stockStatus
      }
    );

    alert(
      'Item saved successfully!'
    );

  }

  clearForm();

};

const editItem = (i: any) => {

  item.value = {

    itemName:
      i.itemName,

    quantity:
      String(i.quantity),

    category:
      i.category,

    price:
      String(i.price),

    stockStatus:
      i.stockStatus

  };

  editingId.value =
    i.id;

  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  });

};

const deleteItem = async (
  id: string
) => {

  const confirmDelete =
    confirm(
      'Are you sure you want to delete this item?'
    );

  if (!confirmDelete) {

    return;

  }

  const itemReference =
    dbRef(
      db,
      `inventory/${id}`
    );

  await remove(
    itemReference
  );

  alert(
    'Item deleted successfully!'
  );

  if (
    editingId.value === id
  ) {

    clearForm();

  }

};

const cancelEdit = () => {

  clearForm();

};

const clearForm = () => {

  item.value = {

    itemName: '',
    quantity: '',
    category: '',
    price: '',
    stockStatus: ''

  };

  editingId.value = null;

};

</script>

<style scoped>

.container {
  width: 100%;
  max-width: 900px;
  margin: auto;
  padding: 25px 18px 50px;
}

.header {
  text-align: center;
  margin-bottom: 25px;
}

.header h1 {
  font-size: 30px;
  font-weight: 700;
  margin-bottom: 5px;
}

.header p {
  margin-top: 0;
  color: #777;
}

.form-card {
  border-radius: 15px;
  margin: 0;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
}

.form-card ion-item {
  margin-bottom: 12px;
}

.button-container {
  margin-top: 20px;
}

.inventory-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 35px;
  margin-bottom: 15px;
}

.inventory-header h2 {
  margin: 0;
  font-size: 24px;
}

.inventory-header p {
  margin: 5px 0 0;
  color: #777;
}

.empty {
  text-align: center;
  padding: 30px;
}

.empty h3 {
  margin-bottom: 5px;
}

.empty p {
  color: #777;
}

.inventory-card {
  background: white;
  border-radius: 15px;
  padding: 20px;
  margin-bottom: 15px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
}

.item-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 15px;
}

.item-header h3 {
  margin: 0 0 7px;
  font-size: 20px;
}

.category {
  display: inline-block;
  font-size: 13px;
  padding: 5px 10px;
  border-radius: 20px;
  background: #eeeeee;
  color: #555;
}

.status {
  font-size: 12px;
  font-weight: 600;
  padding: 6px 10px;
  border-radius: 20px;
  white-space: nowrap;
}

.status-in {
  background: #dff6e4;
  color: #218838;
}

.status-low {
  background: #fff3cd;
  color: #856404;
}

.status-out {
  background: #f8d7da;
  color: #842029;
}

.details {
  display: flex;
  gap: 50px;
  margin-top: 20px;
  padding-top: 15px;
  border-top: 1px solid #eeeeee;
}

.detail {
  display: flex;
  flex-direction: column;
}

.label {
  font-size: 12px;
  color: #888;
  margin-bottom: 4px;
}

.detail strong {
  font-size: 17px;
}

.actions {
  display: flex;
  gap: 8px;
  margin-top: 20px;
}

.actions ion-button {
  margin: 0;
}

@media (max-width: 600px) {

  .container {
    padding: 20px 12px 40px;
  }

  .header h1 {
    font-size: 25px;
  }

  .item-header {
    flex-direction: column;
  }

  .details {
    gap: 30px;
  }

}

</style>