<script setup>
import { ref } from 'vue'

const currentTab = ref('applicant')
const successMessage = ref('')

const applicantForm = ref({
  firstname: '',
  surname: '',
  dob: '',
  address: '',
  phoneCountryCode: '+1',
  phone: '',
  gender: '',
  nationalIdCountry: 'USA',
  nationalId: ''
})

const countries = [
  { code: 'USA', name: 'United States', phoneCode: '+1', phonePattern: /^\d{10}$/, idPattern: /^\d{9}$/, idExample: '9 digits (SSN format)' },
  { code: 'UK', name: 'United Kingdom', phoneCode: '+44', phonePattern: /^\d{10,11}$/, idPattern: /^[A-Z]{2}\d{6}[A-Z]?$/, idExample: '2 letters + 6 digits (e.g., AB123456)' },
  { code: 'CAN', name: 'Canada', phoneCode: '+1', phonePattern: /^\d{10}$/, idPattern: /^\d{9}$/, idExample: '9 digits (SIN format)' },
  { code: 'AUS', name: 'Australia', phoneCode: '+61', phonePattern: /^\d{9,10}$/, idPattern: /^\d{9}$/, idExample: '9 digits (TFN format)' },
  { code: 'DEU', name: 'Germany', phoneCode: '+49', phonePattern: /^\d{10,11}$/, idPattern: /^\d{11}$/, idExample: '11 digits' },
  { code: 'FRA', name: 'France', phoneCode: '+33', phonePattern: /^\d{9}$/, idPattern: /^\d{13}$/, idExample: '13 digits (INSEE format)' },
  { code: 'ESP', name: 'Spain', phoneCode: '+34', phonePattern: /^\d{9}$/, idPattern: /^[XYZ]\d{7}[A-Z]$/, idExample: '1 letter + 7 digits + 1 letter (DNI format)' },
  { code: 'ITA', name: 'Italy', phoneCode: '+39', phonePattern: /^\d{9,10}$/, idPattern: /^[A-Z]{6}\d{2}[A-Z]\d{2}[A-Z]\d{3}[A-Z]$/, idExample: '16 characters (Codice Fiscale format)' },
  { code: 'JPN', name: 'Japan', phoneCode: '+81', phonePattern: /^\d{10,11}$/, idPattern: /^\d{12}$/, idExample: '12 digits (My Number format)' },
  { code: 'CHN', name: 'China', phoneCode: '+86', phonePattern: /^\d{11}$/, idPattern: /^\d{18}$/, idExample: '18 digits' },
  { code: 'IND', name: 'India', phoneCode: '+91', phonePattern: /^\d{10}$/, idPattern: /^[A-Z]{5}\d{4}[A-Z]{1}$/, idExample: '5 letters + 4 digits + 1 letter (PAN format)' },
  { code: 'BRA', name: 'Brazil', phoneCode: '+55', phonePattern: /^\d{10,11}$/, idPattern: /^\d{11}$/, idExample: '11 digits (CPF format)' },
  { code: 'MEX', name: 'Mexico', phoneCode: '+52', phonePattern: /^\d{10}$/, idPattern: /^\d{13}$/, idExample: '13 digits (CURP format)' },
  { code: 'ZAF', name: 'South Africa', phoneCode: '+27', phonePattern: /^\d{9,10}$/, idPattern: /^\d{13}$/, idExample: '13 digits (ID format)' },
  { code: 'NGA', name: 'Nigeria', phoneCode: '+234', phonePattern: /^\d{10,11}$/, idPattern: /^\d{11}$/, idExample: '11 digits (NIN format)' },
  { code: 'RWA', name: 'Rwanda', phoneCode: '+250', phonePattern: /^\d{9}$/, idPattern: /^\d{16}$/, idExample: '16 digits' },
  { code: 'ZWE', name: 'Zimbabwe', phoneCode: '+263', phonePattern: /^\d{9,10}$/, idPattern: /^\d{9}[A-Z]{2}\d{2}$/, idExample: '9 digits + 2 letters + 2 digits' },
  { code: 'KEN', name: 'Kenya', phoneCode: '+254', phonePattern: /^\d{9,10}$/, idPattern: /^\d{8}$/, idExample: '8 digits (ID format)' },
  { code: 'ETH', name: 'Ethiopia', phoneCode: '+251', phonePattern: /^\d{9}$/, idPattern: /^\d{10}$/, idExample: '10 digits' },
  { code: 'EGY', name: 'Egypt', phoneCode: '+20', phonePattern: /^\d{10}$/, idPattern: /^\d{14}$/, idExample: '14 digits (National ID format)' },
  { code: 'MAR', name: 'Morocco', phoneCode: '+212', phonePattern: /^\d{9}$/, idPattern: /^\d{18}$/, idExample: '18 digits (CIN format)' },
  { code: 'TZA', name: 'Tanzania', phoneCode: '+255', phonePattern: /^\d{9,10}$/, idPattern: /^\d{20}$/, idExample: '20 digits (NIN format)' },
  { code: 'UGA', name: 'Uganda', phoneCode: '+256', phonePattern: /^\d{9}$/, idPattern: /^\d{14}$/, idExample: '14 digits (NIN format)' },
  { code: 'GHA', name: 'Ghana', phoneCode: '+233', phonePattern: /^\d{9,10}$/, idPattern: /^\d{15}$/, idExample: '15 digits (Ghana Card format)' },
  { code: 'CMR', name: 'Cameroon', phoneCode: '+237', phonePattern: /^\d{9}$/, idPattern: /^\d{9}$/, idExample: '9 digits' },
  { code: 'COD', name: 'DR Congo', phoneCode: '+243', phonePattern: /^\d{9}$/, idPattern: /^\d{13}$/, idExample: '13 digits' },
  { code: 'SEN', name: 'Senegal', phoneCode: '+221', phonePattern: /^\d{9}$/, idPattern: /^\d{13}$/, idExample: '13 digits (CNI format)' },
  { code: 'CIV', name: 'Ivory Coast', phoneCode: '+225', phonePattern: /^\d{10}$/, idPattern: /^\d{12}$/, idExample: '12 digits' },
  { code: 'ZMB', name: 'Zambia', phoneCode: '+260', phonePattern: /^\d{9}$/, idPattern: /^\d{9}$/, idExample: '9 digits (NRC format)' }
]

const applications = ref([])
const phoneError = ref('')
const nationalIdError = ref('')

function validatePhone(phone, countryCode) {
  const country = countries.find(c => c.phoneCode === countryCode)
  if (!country) return true
  
  const cleanPhone = phone.replace(/[^\d]/g, '')
  if (!country.phonePattern.test(cleanPhone)) {
    phoneError.value = `Invalid phone number for ${country.name}. Expected format: ${cleanPhone.length} digits`
    return false
  }
  phoneError.value = ''
  return true
}

function validateNationalId(id, countryCode) {
  const country = countries.find(c => c.code === countryCode)
  if (!country) return true
  
  if (!country.idPattern.test(id)) {
    nationalIdError.value = `Invalid National ID for ${country.name}. Expected format: ${country.idExample}`
    return false
  }
  nationalIdError.value = ''
  return true
}

function getSelectedCountry() {
  return countries.find(c => c.phoneCode === applicantForm.value.phoneCountryCode)
}

function getSelectedIdCountry() {
  return countries.find(c => c.code === applicantForm.value.nationalIdCountry)
}

function submitApplication() {
  const phoneValid = validatePhone(applicantForm.value.phone, applicantForm.value.phoneCountryCode)
  const idValid = validateNationalId(applicantForm.value.nationalId, applicantForm.value.nationalIdCountry)
  
  if (!phoneValid || !idValid) {
    return
  }
  
  applications.value.push({ ...applicantForm.value })
  successMessage.value = 'Application submitted successfully!'
  
  applicantForm.value = {
    firstname: '',
    surname: '',
    dob: '',
    address: '',
    phoneCountryCode: '+1',
    phone: '',
    gender: '',
    nationalIdCountry: 'USA',
    nationalId: ''
  }
  phoneError.value = ''
  nationalIdError.value = ''

  setTimeout(() => {
    successMessage.value = ''
  }, 3000)
}

function deleteApplication(index) {
  applications.value.splice(index, 1)
}
</script>

<template>
  <div class="app-container">
    <h1>Job Application Dashboard</h1>
    
    <div class="tab-navigation">
      <button 
        :class="{ active: currentTab === 'applicant' }"
        @click="currentTab = 'applicant'"
      >
        Applicant Form
      </button>
      <button 
        :class="{ active: currentTab === 'admin' }"
        @click="currentTab = 'admin'"
      >
        Administrator Dashboard
      </button>
    </div>

    <div v-if="currentTab === 'applicant'" class="tab-content">
      <div v-if="successMessage" class="success-message">
        {{ successMessage }}
      </div>

      <form @submit.prevent="submitApplication" class="applicant-form">
        <h2>Applicant Information</h2>
        
        <div class="form-row">
          <div class="form-group">
            <label for="firstname">Firstname</label>
            <input 
              id="firstname"
              v-model.trim="applicantForm.firstname" 
              type="text" 
              placeholder="Enter firstname"
              required
            />
          </div>
          
          <div class="form-group">
            <label for="surname">Surname</label>
            <input 
              id="surname"
              v-model.trim="applicantForm.surname" 
              type="text" 
              placeholder="Enter surname"
              required
            />
          </div>
        </div>

        <div class="form-group">
          <label for="dob">Date of Birth</label>
          <input 
            id="dob"
            v-model="applicantForm.dob" 
            type="date" 
            required
          />
        </div>

        <div class="form-group">
          <label for="address">Address</label>
          <textarea 
            id="address"
            v-model.trim="applicantForm.address" 
            placeholder="Enter your full address"
            rows="3"
            required
          ></textarea>
        </div>

        <div class="form-group">
          <label for="phone">Phone Number</label>
          <div class="phone-input-group">
            <select 
              v-model="applicantForm.phoneCountryCode"
              class="country-code-select"
              @change="validatePhone(applicantForm.phone, applicantForm.phoneCountryCode)"
            >
              <option v-for="country in countries" :key="country.code" :value="country.phoneCode">
                {{ country.phoneCode }} ({{ country.code }})
              </option>
            </select>
            <input 
              id="phone"
              v-model.trim="applicantForm.phone" 
              type="tel" 
              placeholder="Enter phone number"
              required
              @blur="validatePhone(applicantForm.phone, applicantForm.phoneCountryCode)"
              @input="phoneError = ''"
            />
          </div>
          <div v-if="phoneError" class="error-message">{{ phoneError }}</div>
          <div v-if="getSelectedCountry()" class="hint-message">
            Format: {{ getSelectedCountry().phonePattern.toString().replace(/\^|\$/g, '') }} digits
          </div>
        </div>

        <div class="form-group">
          <label>Gender</label>
          <div class="radio-group">
            <label class="radio-label">
              <input 
                v-model="applicantForm.gender" 
                type="radio" 
                value="Male"
                required
              />
              Male
            </label>
            <label class="radio-label">
              <input 
                v-model="applicantForm.gender" 
                type="radio" 
                value="Female"
              />
              Female
            </label>
            <label class="radio-label">
              <input 
                v-model="applicantForm.gender" 
                type="radio" 
                value="Other"
              />
              Other
            </label>
          </div>
        </div>

        <div class="form-group">
          <label for="nationalId">National ID</label>
          <div class="national-id-group">
            <select 
              v-model="applicantForm.nationalIdCountry"
              class="country-select"
              @change="validateNationalId(applicantForm.nationalId, applicantForm.nationalIdCountry)"
            >
              <option v-for="country in countries" :key="country.code" :value="country.code">
                {{ country.name }}
              </option>
            </select>
            <input 
              id="nationalId"
              v-model.trim="applicantForm.nationalId" 
              type="text" 
              placeholder="Enter national ID"
              required
              @blur="validateNationalId(applicantForm.nationalId, applicantForm.nationalIdCountry)"
              @input="nationalIdError = ''"
            />
          </div>
          <div v-if="nationalIdError" class="error-message">{{ nationalIdError }}</div>
          <div v-if="getSelectedIdCountry()" class="hint-message">
            Format: {{ getSelectedIdCountry().idExample }}
          </div>
        </div>

        <button type="submit" class="submit-btn">Submit Application</button>
      </form>
    </div>

    <div v-if="currentTab === 'admin'" class="tab-content">
      <h2>Administrator Dashboard</h2>
      
      <div v-if="applications.length === 0" class="no-applications">
        <p>No applicants yet</p>
      </div>

      <div v-else class="applications-table">
        <table>
          <thead>
            <tr>
              <th>Firstname</th>
              <th>Surname</th>
              <th>Date of Birth</th>
              <th>Address</th>
              <th>Phone</th>
              <th>Gender</th>
              <th>National ID</th>
              <th>Country</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(app, index) in applications" :key="index">
              <td>{{ app.firstname }}</td>
              <td>{{ app.surname }}</td>
              <td>{{ app.dob }}</td>
              <td>{{ app.address }}</td>
              <td>{{ app.phoneCountryCode }} {{ app.phone }}</td>
              <td>{{ app.gender }}</td>
              <td>{{ app.nationalId }}</td>
              <td>{{ app.nationalIdCountry }}</td>
              <td>
                <button @click="deleteApplication(index)" class="delete-btn">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  max-width: 1000px;
  margin: 30px auto;
  padding: 30px;
  background-color: #f8f9fa;
  border-radius: 12px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 30px;
  font-size: 2rem;
}

h2 {
  color: #34495e;
  margin-bottom: 20px;
  font-size: 1.5rem;
}

.tab-navigation {
  display: flex;
  gap: 10px;
  margin-bottom: 30px;
  justify-content: center;
}

.tab-navigation button {
  padding: 12px 30px;
  background-color: #e0e0e0;
  color: #555;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.tab-navigation button:hover {
  background-color: #d0d0d0;
}

.tab-navigation button.active {
  background-color: #42b883;
  color: white;
}

.tab-content {
  background-color: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.success-message {
  background-color: #d4edda;
  color: #155724;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  text-align: center;
  font-weight: 600;
  border: 1px solid #c3e6cb;
}

.applicant-form {
  max-width: 600px;
  margin: 0 auto;
}

.form-row {
  display: flex;
  gap: 20px;
}

.form-row .form-group {
  flex: 1;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #2c3e50;
  font-size: 0.95rem;
}

input[type="text"],
input[type="date"],
input[type="tel"],
textarea,
select {
  width: 100%;
  padding: 12px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s ease;
  box-sizing: border-box;
  background-color: white;
  cursor: pointer;
}

input:focus,
textarea:focus,
select:focus {
  border-color: #42b883;
  outline: none;
}

textarea {
  resize: vertical;
  font-family: inherit;
}

.radio-group {
  display: flex;
  gap: 20px;
  margin-top: 8px;
}

.radio-label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-weight: normal;
  cursor: pointer;
}

.radio-label input[type="radio"] {
  width: auto;
  cursor: pointer;
}

.submit-btn {
  width: 100%;
  padding: 14px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  font-weight: 700;
  cursor: pointer;
  transition: background-color 0.3s ease;
  margin-top: 10px;
}

.submit-btn:hover {
  background-color: #35495e;
}

.error-message {
  background-color: #f8d7da;
  color: #721c24;
  padding: 10px;
  border-radius: 6px;
  margin-top: 8px;
  font-size: 0.9rem;
  border: 1px solid #f5c6cb;
}

.hint-message {
  color: #6c757d;
  font-size: 0.85rem;
  margin-top: 6px;
  font-style: italic;
}

.delete-btn {
  padding: 8px 16px;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.delete-btn:hover {
  background-color: #c82333;
}

.phone-input-group,
.national-id-group {
  display: flex;
  gap: 10px;
}

.country-code-select {
  width: 140px;
  flex-shrink: 0;
}

.country-select {
  width: 200px;
  flex-shrink: 0;
}

.phone-input-group input,
.national-id-group input {
  flex: 1;
}

.no-applications {
  text-align: center;
  padding: 60px 20px;
  color: #7f8c8d;
  font-size: 1.2rem;
  background-color: #f8f9fa;
  border-radius: 8px;
  border: 2px dashed #e0e0e0;
}

.applications-table {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

thead {
  background-color: #42b883;
  color: white;
}

th {
  padding: 15px;
  text-align: left;
  font-weight: 600;
  font-size: 0.95rem;
}

tbody tr {
  border-bottom: 1px solid #e0e0e0;
}

tbody tr:hover {
  background-color: #f8f9fa;
}

td {
  padding: 15px;
  color: #2c3e50;
  font-size: 0.95rem;
}

tbody tr:last-child {
  border-bottom: none;
}
</style>

