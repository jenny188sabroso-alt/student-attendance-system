
<script setup>
import { ref, computed, onMounted } from 'vue'

const STORAGE_KEY = 'cpsu-attendance-demo'

const defaultStudent = {
  fullName: 'Rodriguez Janet P.',
  studentId: '2026-0001',
  email: 'student@cpsu.edu.ph',
  college: 'College of Computer Studies',
  course: 'BS Information Technology',
  yearLevel: '4th Year',
  status: 'Active'
}

const events = ref([
  {
    id: 1,
    name: 'University Foundation Day',
    date: '2026-11-20',
    time: '8:00 AM',
    location: 'CPSU Main Campus'
  },
  {
    id: 2,
    name: 'Student Leadership Seminar',
    date: '2026-11-25',
    time: '9:00 AM',
    location: 'University Auditorium'
  },
  {
    id: 3,
    name: 'University Sports Festival',
    date: '2026-12-05',
    time: '7:30 AM',
    location: 'University Gymnasium'
  },
  {
    id: 4,
    name: 'University Research Conference',
    date: '2026-12-12',
    time: '8:30 AM',
    location: 'CPSU Conference Hall'
  }
])

const defaultAttendance = [
  {
    id: 101,
    event: 'University Orientation',
    date: '2026-10-15',
    time: '8:30 AM',
    status: 'Present'
  },
  {
    id: 102,
    event: 'Student General Assembly',
    date: '2026-10-20',
    time: '9:00 AM',
    status: 'Present'
  }
]

function loadSavedData() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY)) || {}
  } catch {
    return {}
  }
}

const saved = loadSavedData()

const student = ref({
  ...defaultStudent,
  ...(saved.student || {})
})

const attendance = ref(saved.attendance || defaultAttendance)
const registeredAccounts = ref(saved.accounts || [
  {
    email: 'student@cpsu.edu.ph',
    password: 'Student123!'
  }
])

const currentPage = ref('login')
const isLoggedIn = ref(false)
const showPassword = ref(false)
const showRegisterPassword = ref(false)
const editingProfile = ref(false)
const loginLoading = ref(false)
const notification = ref('')
const notificationType = ref('success')

const loginForm = ref({
  email: 'student@cpsu.edu.ph',
  password: 'Student123!'
})

const registerForm = ref({
  fullName: '',
  studentId: '',
  email: '',
  password: '',
  confirmPassword: ''
})

const profileForm = ref({ ...student.value })

function saveData() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify({
    student: student.value,
    attendance: attendance.value,
    accounts: registeredAccounts.value
  }))
}

function notify(message, type = 'success') {
  notification.value = message
  notificationType.value = type

  setTimeout(() => {
    notification.value = ''
  }, 3000)
}

const firstName = computed(() => {
  return student.value.fullName.trim().split(' ')[0] || 'Student'
})

const initials = computed(() => {
  return student.value.fullName
    .split(' ')
    .filter(Boolean)
    .slice(0, 2)
    .map(word => word[0].toUpperCase())
    .join('')
})

const attendedCount = computed(() => attendance.value.length)

const attendanceRate = computed(() => {
  const total = attendedCount.value
  const rate = events.value.length + total

  return rate ? Math.round((total / rate) * 100) : 0
})

const upcomingEvents = computed(() => {
  const today = new Date().toISOString().slice(0, 10)

  return events.value.filter(event => event.date >= today)
})

function login() {
  const email = loginForm.value.email.trim().toLowerCase()
  const password = loginForm.value.password

  if (!email || !password) {
    notify('Please enter your email and password.', 'error')
    return
  }

  loginLoading.value = true

  setTimeout(() => {
    const account = registeredAccounts.value.find(
      account =>
        account.email.toLowerCase() === email &&
        account.password === password
    )

    loginLoading.value = false

    if (!account) {
      notify('Invalid email or password.', 'error')
      return
    }

    isLoggedIn.value = true
    currentPage.value = 'dashboard'
    notify('Welcome to your CPSU student portal!')
  }, 600)
}

function register() {
  const form = registerForm.value

  if (!form.fullName || !form.studentId || !form.email ||
      !form.password || !form.confirmPassword) {
    notify('Please complete all registration fields.', 'error')
    return
  }

  if (form.password.length < 8) {
    notify('Password must contain at least 8 characters.', 'error')
    return
  }

  if (form.password !== form.confirmPassword) {
    notify('Passwords do not match.', 'error')
    return
  }

  const exists = registeredAccounts.value.some(
    account => account.email.toLowerCase() === form.email.toLowerCase()
  )

  if (exists) {
    notify('This email is already registered.', 'error')
    return
  }

  registeredAccounts.value.push({
    email: form.email,
    password: form.password
  })

  student.value = {
    ...defaultStudent,
    fullName: form.fullName,
    studentId: form.studentId,
    email: form.email
  }

  saveData()

  loginForm.value.email = form.email
  loginForm.value.password = ''

  registerForm.value = {
    fullName: '',
    studentId: '',
    email: '',
    password: '',
    confirmPassword: ''
  }

  currentPage.value = 'login'
  notify('Registration successful! You can now log in.')
}

function openProfile() {
  profileForm.value = { ...student.value }
  editingProfile.value = false
  currentPage.value = 'profile'
}

function editProfile() {
  profileForm.value = { ...student.value }
  editingProfile.value = true
}

function saveProfile() {
  if (!profileForm.value.fullName.trim() ||
      !profileForm.value.email.trim()) {
    notify('Name and email cannot be empty.', 'error')
    return
  }

  const emailChanged =
    profileForm.value.email.toLowerCase() !== student.value.email.toLowerCase()

  if (emailChanged && registeredAccounts.value.some(
    account => account.email.toLowerCase() ===
      profileForm.value.email.toLowerCase()
  )) {
    notify('That email address is already in use.', 'error')
    return
  }

  if (emailChanged) {
    const oldAccount = registeredAccounts.value.find(
      account => account.email.toLowerCase() ===
        student.value.email.toLowerCase()
    )

    if (oldAccount) oldAccount.email = profileForm.value.email
    loginForm.value.email = profileForm.value.email
  }

  student.value = { ...profileForm.value }
  saveData()
  editingProfile.value = false

  notify('Your profile has been updated successfully!')
}

function cancelEdit() {
  profileForm.value = { ...student.value }
  editingProfile.value = false
}

function checkIn(event) {
  const alreadyAttended = attendance.value.some(
    record => record.id === event.id
  )

  if (alreadyAttended) {
    notify('You already have an attendance record for this event.', 'error')
    return
  }

  attendance.value.unshift({
    id: event.id,
    event: event.name,
    date: event.date,
    time: new Date().toLocaleTimeString([], {
      hour: '2-digit',
      minute: '2-digit'
    }),
    status: 'Present'
  })

  saveData()
  notify('Demo check-in recorded successfully!')
}

function logout() {
  isLoggedIn.value = false
  currentPage.value = 'login'
  editingProfile.value = false
  loginForm.value.password = ''
  notify('You have successfully logged out.')
}

function formatDate(date) {
  return new Date(date + 'T00:00:00').toLocaleDateString(
    'en-US',
    { month: 'long', day: 'numeric', year: 'numeric' }
  )
}

onMounted(() => {
  if (saved.student) {
    student.value = { ...defaultStudent, ...saved.student }
  }
})
</script>

<template>
  <div class="app">

    <!-- NOTIFICATION -->
    <Transition name="fade">
      <div
        v-if="notification"
        class="notification"
        :class="notificationType"
      >
        {{ notification }}
      </div>
    </Transition>

    <!-- LOGIN -->
    <section v-if="!isLoggedIn && currentPage === 'login'"
      class="auth-page">

      <div class="auth-card">
        <div class="brand-logo">
          <img src="/cpsu-logo.png" alt="CPSU Logo"
            @error="$event.target.style.display='none'">
          <div class="logo-fallback">CPSU</div>
        </div>

        <h1>Student Attendance</h1>
        <p class="subtitle">Monitoring System</p>

        <form @submit.prevent="login">
          <div class="form-group">
            <label>University Email</label>
            <input
              v-model="loginForm.email"
              type="email"
              placeholder="Enter your university email"
              required
            >
          </div>

          <div class="form-group">
            <label>Password</label>

            <div class="password-field">
              <input
                v-model="loginForm.password"
                :type="showPassword ? 'text' : 'password'"
                placeholder="Enter your password"
                required
              >

              <button
                type="button"
                class="eye-button"
                @click="showPassword = !showPassword"
                :aria-label="showPassword ? 'Hide password' : 'Show password'"
              >
                {{ showPassword ? 'Hide' : 'Show' }}
              </button>
            </div>
          </div>

          <div class="remember-row">
            <label class="remember-label">
              <input type="checkbox">
              Remember me
            </label>
            <span class="forgot-text">CPSU Student Portal</span>
          </div>

          <button class="primary-button" :disabled="loginLoading">
            {{ loginLoading ? 'Signing in...' : 'Login to Account' }}
          </button>
        </form>

        <p class="auth-switch">
          Don't have an account?
          <a @click="currentPage = 'register'">Register here</a>
        </p>

        <div class="demo-box">
          <strong>Demo Student Account</strong>
          <p>Email: student@cpsu.edu.ph</p>
          <p>Password: Student123!</p>
        </div>

        <p class="auth-footer">
          © 2026 Central Philippines State University
        </p>
      </div>
    </section>

    <!-- REGISTRATION -->
    <section v-else-if="!isLoggedIn && currentPage === 'register'"
      class="auth-page">

      <div class="auth-card register-card">
        <div class="brand-logo">
          <img src="/cpsu-logo.png" alt="CPSU Logo">
          <div class="logo-fallback">CPSU</div>
        </div>

        <h1>Create Student Account</h1>
        <p class="subtitle">Register for the CPSU Attendance Portal</p>

        <form @submit.prevent="register">
          <div class="form-group">
            <label>Complete Name</label>
            <input v-model="registerForm.fullName"
              placeholder="Enter your complete name" required>
          </div>

          <div class="form-group">
            <label>Student ID</label>
            <input v-model="registerForm.studentId"
              placeholder="e.g. 2026-0001" required>
          </div>

          <div class="form-group">
            <label>University Email</label>
            <input v-model="registerForm.email" type="email"
              placeholder="Enter your university email" required>
          </div>

          <div class="form-group">
            <label>Password</label>
            <div class="password-field">
              <input v-model="registerForm.password"
                :type="showRegisterPassword ? 'text' : 'password'"
                placeholder="At least 8 characters" required>
              <button type="button" class="eye-button"
                @click="showRegisterPassword = !showRegisterPassword">
                {{ showRegisterPassword ? 'Hide' : 'Show' }}
              </button>
            </div>
          </div>

          <div class="form-group">
            <label>Confirm Password</label>
            <input v-model="registerForm.confirmPassword"
              type="password" placeholder="Confirm your password" required>
          </div>

          <button class="primary-button">Create Account</button>
        </form>

        <p class="auth-switch">
          Already registered?
          <a @click="currentPage = 'login'">Back to Login</a>
        </p>
      </div>
    </section>

    <!-- MAIN STUDENT PORTAL -->
    <div v-else class="portal">

      <!-- SIDEBAR -->
      <aside class="sidebar">
        <div class="sidebar-brand">
          <img src="/cpsu-logo.png" alt="CPSU"
            @error="$event.target.style.display='none'">
          <div>
            <h2>CPSU</h2>
            <span>Attendance System</span>
          </div>
        </div>

        <p class="menu-label">MAIN MENU</p>

        <button
          v-for="item in [
            {id:'dashboard', icon:'▦', label:'Dashboard'},
            {id:'events', icon:'▣', label:'University Events'},
            {id:'qr', icon:'▦', label:'My QR Code'},
            {id:'history', icon:'✓', label:'Attendance History'},
            {id:'profile', icon:'◉', label:'My Profile'}
          ]"
          :key="item.id"
          class="nav-link"
          :class="{active: currentPage === item.id}"
          @click="item.id === 'profile' ? openProfile() : currentPage = item.id"
        >
          <span class="nav-icon">{{ item.icon }}</span>
          {{ item.label }}
        </button>

        <div class="sidebar-bottom">
          <button class="logout-button" @click="logout">
            ↪ Logout
          </button>
        </div>
      </aside>

      <!-- MAIN CONTENT -->
      <main class="main-content">

        <!-- TOP HEADER -->
        <header class="topbar">
          <div>
            <h2>
              {{
                {
                  dashboard:'Dashboard',
                  events:'University Events',
                  qr:'My QR Code',
                  history:'Attendance History',
                  profile:'My Profile'
                }[currentPage]
              }}
            </h2>
            <p>Central Philippines State University</p>
          </div>

          <div class="user-info">
            <div class="avatar">{{ initials }}</div>
            <div>
              <strong>{{ student.fullName }}</strong>
              <span>Student</span>
            </div>
          </div>
        </header>

        <!-- DASHBOARD -->
        <section v-if="currentPage === 'dashboard'">

          <div class="welcome-section">
            <div>
              <h1>Welcome back, {{ firstName }}!</h1>
              <p>Here's your university attendance overview.</p>
            </div>
            <span class="date-label">
              {{ new Date().toLocaleDateString('en-US',
                {month:'long',day:'numeric',year:'numeric'}) }}
            </span>
          </div>

          <!-- STATISTICS -->
          <div class="stats-grid">
            <div class="stat-card">
              <div class="stat-icon blue">▣</div>
              <p>University Events</p>
              <h2>{{ events.length }}</h2>
              <span>Available events</span>
            </div>

            <div class="stat-card">
              <div class="stat-icon green">✓</div>
              <p>Events Attended</p>
              <h2>{{ attendedCount }}</h2>
              <span>Recorded attendance</span>
            </div>

            <div class="stat-card">
              <div class="stat-icon purple">%</div>
              <p>Attendance Rate</p>
              <h2>{{ attendanceRate }}%</h2>
              <span>Based on demo records</span>
            </div>

            <div class="stat-card">
              <div class="stat-icon orange">◉</div>
              <p>Student Status</p>
              <h2 class="active-status">{{ student.status }}</h2>
              <span>Account status</span>
            </div>
          </div>

          <!-- UPCOMING EVENTS -->
          <div class="content-card">
            <div class="section-heading">
              <div>
                <h2>Upcoming University Events</h2>
                <p>Stay informed about campus activities.</p>
              </div>
              <button class="text-button" @click="currentPage='events'">
                View All →
              </button>
            </div>

            <div v-if="upcomingEvents.length">
              <div v-for="event in upcomingEvents.slice(0,3)"
                :key="event.id" class="event-row">
                <div class="event-date">
                  <strong>{{ event.date.slice(8,10) }}</strong>
                  <span>{{ new Date(event.date + 'T00:00:00')
                    .toLocaleDateString('en-US',{month:'short'}) }}</span>
                </div>
                <div class="event-details">
                  <strong>{{ event.name }}</strong>
                  <span>{{ formatDate(event.date) }} · {{ event.location }}</span>
                </div>
                <span class="status-badge upcoming">Upcoming</span>
              </div>
            </div>
            <p v-else class="empty-state">No upcoming events.</p>
          </div>

          <!-- RECENT ATTENDANCE -->
          <div class="content-card">
            <div class="section-heading">
              <div>
                <h2>Recent Attendance</h2>
                <p>Your latest event participation records.</p>
              </div>
              <button class="text-button" @click="currentPage='history'">
                View History →
              </button>
            </div>

            <div v-if="attendance.length" class="table-wrapper">
              <table>
                <thead>
                  <tr>
                    <th>University Event</th>
                    <th>Date</th>
                    <th>Time</th>
                    <th>Status</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="record in attendance.slice(0,5)"
                    :key="record.id">
                    <td>{{ record.event }}</td>
                    <td>{{ formatDate(record.date) }}</td>
                    <td>{{ record.time }}</td>
                    <td><span class="status-badge present">
                      {{ record.status }}</span></td>
                  </tr>
                </tbody>
              </table>
            </div>
            <p v-else class="empty-state">No attendance records yet.</p>
          </div>

          <!-- QUICK ACTIONS -->
          <div class="quick-actions">
            <button @click="currentPage='qr'">▦ View My QR Code</button>
            <button @click="openProfile()">◉ Edit My Profile</button>
            <button @click="currentPage='events'">▣ Explore Events</button>
          </div>
        </section>

        <!-- UNIVERSITY EVENTS -->
        <section v-else-if="currentPage === 'events'">
          <div class="page-intro">
            <h1>University Events</h1>
            <p>View available university activities and schedules.</p>
          </div>

          <div class="event-grid">
            <div v-for="event in events" :key="event.id"
              class="content-card event-card">
              <div class="event-card-icon">▣</div>
              <h3>{{ event.name }}</h3>
              <p>📅 {{ formatDate(event.date) }}</p>
              <p>◷ {{ event.time }}</p>
              <p>📍 {{ event.location }}</p>

              <button
                class="primary-button small-button"
                :disabled="attendance.some(a => a.id === event.id)"
                @click="checkIn(event)"
              >
                {{
                  attendance.some(a => a.id === event.id)
                    ? 'Already Checked In'
                    : 'Demo Check-in'
                }}
              </button>
            </div>
          </div>
        </section>

        <!-- QR CODE -->
        <section v-else-if="currentPage === 'qr'">
          <div class="page-intro">
            <h1>My Student QR Code</h1>
            <p>Your student identification information.</p>
          </div>

          <div class="content-card qr-card">
            <div class="avatar large-avatar">{{ initials }}</div>
            <h2>{{ student.fullName }}</h2>
            <p>{{ student.studentId }}</p>
            <p>{{ student.course }}</p>

            <div class="qr-placeholder">
              <div class="qr-pattern">
                <span v-for="n in 121" :key="n"
                  :class="{dark: ((n * 7 + Math.floor(n / 3)) % 5) < 2}">
                </span>
              </div>
            </div>

            <p class="qr-note">
              QR code preview only. Actual secure QR generation
              and scanning will be connected in the next phase.
            </p>
          </div>
        </section>

        <!-- ATTENDANCE HISTORY -->
        <section v-else-if="currentPage === 'history'">
          <div class="page-intro">
            <h1>Attendance History</h1>
            <p>Review your university event participation.</p>
          </div>

          <div class="content-card">
            <div class="table-wrapper">
              <table>
                <thead>
                  <tr>
                    <th>University Event</th>
                    <th>Date</th>
                    <th>Time</th>
                    <th>Status</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="record in attendance" :key="record.id">
                    <td>{{ record.event }}</td>
                    <td>{{ formatDate(record.date) }}</td>
                    <td>{{ record.time }}</td>
                    <td><span class="status-badge present">
                      {{ record.status }}</span></td>
                  </tr>
                </tbody>
              </table>
            </div>
            <p v-if="!attendance.length" class="empty-state">
              No attendance records found.
            </p>
          </div>
        </section>

        <!-- MY PROFILE -->
        <section v-else-if="currentPage === 'profile'">
          <div class="page-intro">
            <h1>My Student Profile</h1>
            <p>View and manage your registered student information.</p>
          </div>

          <div class="content-card profile-card">

            <div class="profile-heading">
              <div class="avatar profile-avatar">{{ initials }}</div>
              <div>
                <h2>{{ student.fullName }}</h2>
                <p>Student ID: {{ student.studentId }}</p>
                <span class="status-badge present">Active Student</span>
              </div>

              <button v-if="!editingProfile"
                class="primary-button edit-button"
                @click="editProfile">
                ✎ Edit Profile
              </button>
            </div>

            <div v-if="!editingProfile" class="profile-information">
              <div class="info-item">
                <span>Full Name</span>
                <strong>{{ student.fullName }}</strong>
              </div>
              <div class="info-item">
                <span>Student ID</span>
                <strong>{{ student.studentId }}</strong>
              </div>
              <div class="info-item">
                <span>University Email</span>
                <strong>{{ student.email }}</strong>
              </div>
              <div class="info-item">
                <span>College / Department</span>
                <strong>{{ student.college }}</strong>
              </div>
              <div class="info-item">
                <span>Course / Program</span>
                <strong>{{ student.course }}</strong>
              </div>
              <div class="info-item">
                <span>Year Level</span>
                <strong>{{ student.yearLevel }}</strong>
              </div>
            </div>

            <!-- EDIT FORM -->
            <div v-else class="profile-edit-form">
              <div class="edit-heading">
                <h2>Edit Personal Information</h2>
                <p>Update your student profile details below.</p>
              </div>

              <div class="edit-grid">
                <div class="form-group">
                  <label>Full Name</label>
                  <input v-model="profileForm.fullName"
                    placeholder="Enter your full name">
                </div>

                <div class="form-group">
                  <label>Student ID</label>
                  <input v-model="profileForm.studentId"
                    placeholder="Student ID">
                </div>

                <div class="form-group">
                  <label>University Email</label>
                  <input v-model="profileForm.email" type="email">
                </div>

                <div class="form-group">
                  <label>College / Department</label>
                  <input v-model="profileForm.college">
                </div>

                <div class="form-group">
                  <label>Course / Program</label>
                  <input v-model="profileForm.course">
                </div>

                <div class="form-group">
                  <label>Year Level</label>
                  <select v-model="profileForm.yearLevel">
                    <option>1st Year</option>
                    <option>2nd Year</option>
                    <option>3rd Year</option>
                    <option>4th Year</option>
                    <option>5th Year</option>
                  </select>
                </div>
              </div>

              <div class="edit-actions">
                <button class="cancel-button" @click="cancelEdit">
                  Cancel
                </button>
                <button class="primary-button save-button"
                  @click="saveProfile">
                  ✓ Save Changes
                </button>
              </div>
            </div>

          </div>
        </section>

        <!-- FOOTER -->
        <footer class="footer">
          © 2026 Central Philippines State University.
          Student Attendance Monitoring System.
        </footer>

      </main>
    </div>
  </div>
</template>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: Inter, "Segoe UI", Arial, sans-serif;
  background: #f3f6fc;
  color: #15336d;
}

button, input, select {
  font: inherit;
}

button {
  cursor: pointer;
}

.app {
  min-height: 100vh;
}

/* AUTHENTICATION */

.auth-page {
  min-height: 100vh;
  background: linear-gradient(135deg, #0b3186, #1764ed, #0c3b9b);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 30px 16px;
}

.auth-card {
  background: white;
  width: 100%;
  max-width: 530px;
  padding: 38px 42px;
  border-radius: 20px;
  box-shadow: 0 20px 60px #001c6038;
  text-align: center;
}

.brand-logo {
  width: 100px;
  height: 100px;
  margin: 0 auto 15px;
  position: relative;
}

.brand-logo img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  position: relative;
  z-index: 2;
}

.logo-fallback {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  font-weight: 900;
  color: #0d398b;
}

.auth-card h1 {
  color: #103b91;
  font-size: 29px;
  margin-bottom: 8px;
}

.subtitle {
  color: #74839f;
  margin-bottom: 30px;
}

.auth-card form {
  text-align: left;
}

.form-group {
  margin-bottom: 18px;
}

.form-group label {
  display: block;
  font-size: 14px;
  font-weight: 650;
  margin-bottom: 8px;
  color: #142d5b;
}

.form-group input,
.form-group select {
  width: 100%;
  height: 48px;
  border: 1px solid #ccd7e9;
  border-radius: 9px;
  padding: 0 14px;
  outline: none;
  background: #fff;
  color: #17366e;
  transition: .2s;
}

.form-group input:focus,
.form-group select:focus {
  border-color: #1764ed;
  box-shadow: 0 0 0 3px #1764ed17;
}

.password-field {
  display: flex;
  align-items: center;
  position: relative;
}

.password-field input {
  padding-right: 65px;
}

.eye-button {
  position: absolute;
  right: 12px;
  background: transparent;
  border: none;
  color: #1764ed;
  font-size: 12px;
  font-weight: 700;
}

.remember-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 3px 0 20px;
  font-size: 12px;
  color: #687a99;
}

.remember-label {
  display: flex;
  align-items: center;
  gap: 6px;
}

.primary-button {
  background: #1764ed;
  border: none;
  border-radius: 9px;
  padding: 14px 20px;
  color: white;
  font-weight: 700;
  transition: .2s;
}

.primary-button:hover {
  background: #104ab8;
  transform: translateY(-1px);
}

.primary-button:disabled {
  opacity: .65;
  cursor: wait;
}

.auth-card .primary-button {
  width: 100%;
}

.auth-switch {
  margin-top: 22px;
  font-size: 13px;
  color: #536581;
}

.auth-switch a {
  color: #1764ed;
  font-weight: 700;
  cursor: pointer;
}

.demo-box {
  background: #edf3ff;
  padding: 13px;
  border-radius: 9px;
  margin-top: 24px;
  text-align: left;
  font-size: 12px;
  line-height: 1.8;
}

.auth-footer {
  font-size: 11px;
  color: #9aa7bd;
  margin-top: 24px;
}

.register-card {
  max-width: 550px;
}

.register-card .brand-logo {
  width: 75px;
  height: 75px;
}

/* PORTAL LAYOUT */

.portal {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  background: #10377f;
  color: white;
  width: 255px;
  flex-shrink: 0;
  padding: 25px 17px;
  display: flex;
  flex-direction: column;
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 0 8px 35px;
}

.sidebar-brand img {
  width: 53px;
  height: 53px;
  object-fit: contain;
}

.sidebar-brand h2 {
  font-size: 21px;
}

.sidebar-brand span {
  font-size: 11px;
  color: #d4e3ff;
}

.menu-label {
  color: #b8d0ff;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 1px;
  padding: 0 10px;
  margin-bottom: 13px;
}

.nav-link {
  display: flex;
  align-items: center;
  gap: 13px;
  background: transparent;
  border: none;
  color: white;
  text-align: left;
  padding: 14px 12px;
  border-radius: 8px;
  margin-bottom: 8px;
  font-size: 13px;
  transition: .2s;
}

.nav-link:hover,
.nav-link.active {
  background: #245bb9;
}

.nav-icon {
  width: 18px;
  font-size: 18px;
  text-align: center;
}

.sidebar-bottom {
  margin-top: auto;
  padding-top: 35px;
}

.logout-button {
  background: #1c4b9d;
  border: none;
  color: white;
  padding: 14px;
  width: 100%;
  border-radius: 8px;
  text-align: left;
}

.logout-button:hover {
  background: #2a61bc;
}

/* MAIN */

.main-content {
  flex: 1;
  min-width: 0;
  padding: 25px 35px;
  background: #f3f6fc;
}

.topbar {
  background: white;
  padding: 17px 23px;
  border-radius: 13px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  box-shadow: 0 4px 15px #1b37620b;
  margin-bottom: 30px;
}

.topbar h2 {
  font-size: 20px;
}

.topbar p,
.user-info span {
  font-size: 12px;
  color: #8492ab;
  margin-top: 4px;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.user-info strong {
  font-size: 13px;
  display: block;
}

.user-info span {
  display: block;
}

.avatar {
  width: 43px;
  height: 43px;
  background: #e1ebff;
  color: #174da8;
  border-radius: 50%;
  display: grid;
  place-items: center;
  font-size: 15px;
  font-weight: 800;
}

.welcome-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 10px 0 23px;
  gap: 15px;
}

.welcome-section h1,
.page-intro h1 {
  font-size: 25px;
  margin-bottom: 8px;
}

.welcome-section p,
.page-intro p {
  color: #8593aa;
  font-size: 13px;
}

.date-label {
  font-size: 12px;
  color: #71819e;
  background: white;
  padding: 11px 15px;
  border-radius: 8px;
}

/* STATISTICS */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 17px;
  margin-bottom: 23px;
}

.stat-card {
  background: white;
  padding: 21px;
  border-radius: 13px;
  box-shadow: 0 4px 15px #1b37620b;
}

.stat-icon {
  width: 38px;
  height: 38px;
  border-radius: 9px;
  display: grid;
  place-items: center;
  font-size: 19px;
  font-weight: 800;
  margin-bottom: 16px;
}

.blue { background: #e7efff; color: #2362ca; }
.green { background: #e4f8ed; color: #199d5c; }
.purple { background: #f0eaff; color: #8150d5; }
.orange { background: #fff1df; color: #d98520; }

.stat-card p {
  font-size: 12px;
  color: #8794aa;
}

.stat-card h2 {
  font-size: 27px;
  margin: 7px 0;
  color: #174ca4;
}

.stat-card span {
  font-size: 11px;
  color: #9aa6ba;
}

.active-status {
  color: #15975a !important;
  font-size: 22px !important;
}

/* CONTENT CARDS */

.content-card {
  background: white;
  border-radius: 13px;
  padding: 24px;
  margin-bottom: 22px;
  box-shadow: 0 4px 15px #1b37620b;
}

.section-heading {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-bottom: 18px;
}

.section-heading h2 {
  font-size: 17px;
}

.section-heading p {
  font-size: 12px;
  color: #8997ad;
  margin-top: 5px;
}

.text-button {
  background: none;
  border: none;
  color: #1764ed;
  font-weight: 650;
  font-size: 12px;
}

.event-row {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 15px 0;
  border-bottom: 1px solid #edf0f6;
}

.event-row:last-child {
  border-bottom: none;
}

.event-date {
  width: 47px;
  height: 49px;
  flex-shrink: 0;
  background: #edf3ff;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: #1852ac;
}

.event-date strong {
  font-size: 17px;
}

.event-date span {
  font-size: 10px;
}

.event-details {
  flex: 1;
}

.event-details strong {
  font-size: 13px;
  display: block;
}

.event-details span {
  color: #8997ad;
  font-size: 11px;
  display: block;
  margin-top: 5px;
}

.status-badge {
  padding: 6px 11px;
  border-radius: 20px;
  font-size: 10px;
  font-weight: 700;
  white-space: nowrap;
}

.upcoming {
  background: #fff2dd;
  color: #c47b16;
}

.present {
  background: #e2f7eb;
  color: #159456;
}

/* TABLE */

.table-wrapper {
  width: 100%;
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
  font-size: 12px;
  min-width: 540px;
}

th {
  background: #f5f7fb;
  color: #657793;
  font-weight: 650;
}

th, td {
  padding: 14px 12px;
  border-bottom: 1px solid #edf0f6;
}

td {
  color: #25416f;
}

.quick-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 25px;
}

.quick-actions button {
  padding: 13px 17px;
  background: #e8efff;
  color: #1755ba;
  border: none;
  border-radius: 8px;
  font-size: 12px;
  font-weight: 650;
}

/* EVENTS */

.page-intro {
  margin: 15px 0 24px;
}

.event-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 18px;
}

.event-card {
  margin: 0;
}

.event-card-icon {
  width: 43px;
  height: 43px;
  border-radius: 10px;
  display: grid;
  place-items: center;
  background: #e9f0ff;
  color: #1a5fc9;
  font-size: 22px;
  margin-bottom: 16px;
}

.event-card h3 {
  font-size: 16px;
  margin-bottom: 14px;
}

.event-card p {
  color: #8190a9;
  font-size: 12px;
  margin-bottom: 9px;
}

.small-button {
  width: 100%;
  margin-top: 12px;
  padding: 11px;
  font-size: 12px;
}

.small-button:disabled {
  background: #dce5f3;
  color: #667895;
  cursor: not-allowed;
  transform: none;
}

/* QR */

.qr-card {
  text-align: center;
  padding: 40px 20px;
}

.large-avatar {
  width: 78px;
  height: 78px;
  font-size: 25px;
  margin: 0 auto 15px;
}

.qr-card h2 {
  font-size: 20px;
  margin-bottom: 10px;
}

.qr-card > p {
  color: #70829e;
  font-size: 13px;
  margin-bottom: 10px;
}

.qr-placeholder {
  margin: 25px auto;
  width: 190px;
  height: 190px;
  padding: 12px;
  border: 8px solid #f1f4f9;
}

.qr-pattern {
  display: grid;
  grid-template-columns: repeat(11, 1fr);
  height: 100%;
  gap: 2px;
}

.qr-pattern span.dark {
  background: #10377f;
}

.qr-pattern span {
  background: white;
}

.qr-card .qr-note {
  font-size: 11px;
}

/* PROFILE */

.profile-card {
  padding: 32px;
}

.profile-heading {
  display: flex;
  align-items: center;
  gap: 18px;
  padding-bottom: 27px;
  border-bottom: 1px solid #edf0f6;
}

.profile-avatar {
  width: 75px;
  height: 75px;
  font-size: 23px;
  flex-shrink: 0;
}

.profile-heading h2 {
  font-size: 20px;
  margin-bottom: 6px;
}

.profile-heading p {
  color: #8997ad;
  font-size: 12px;
  margin-bottom: 9px;
}

.edit-button {
  margin-left: auto;
  padding: 11px 17px;
  font-size: 12px;
  width: auto;
}

.profile-information {
  max-width: 700px;
  margin: 15px auto 0;
}

.info-item {
  display: flex;
  flex-direction: column;
  gap: 9px;
  padding: 17px 10px;
  border-bottom: 1px solid #edf0f6;
}

.info-item span {
  color: #8594ad;
  font-size: 12px;
}

.info-item strong {
  color: #17386f;
  font-size: 13px;
}

/* PROFILE EDITING */

.profile-edit-form {
  padding-top: 26px;
}

.edit-heading {
  margin-bottom: 22px;
}

.edit-heading h2 {
  font-size: 19px;
  margin-bottom: 7px;
}

.edit-heading p {
  font-size: 12px;
  color: #8997ad;
}

.edit-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 5px 20px;
}

.edit-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 15px;
}

.cancel-button {
  background: #edf1f7;
  color: #536581;
  border: none;
  border-radius: 8px;
  padding: 12px 22px;
}

.save-button {
  width: auto;
  padding: 12px 22px;
}

/* FOOTER */

.footer {
  text-align: center;
  color: #99a5b8;
  font-size: 11px;
  padding: 25px 10px;
}

.empty-state {
  color: #8b98ad;
  font-size: 13px;
  padding: 25px 5px;
  text-align: center;
}

/* NOTIFICATION */

.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 100;
  padding: 15px 22px;
  border-radius: 9px;
  color: white;
  font-size: 13px;
  box-shadow: 0 5px 20px #0002;
}

.notification.success {
  background: #159456;
}

.notification.error {
  background: #d84343;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity .25s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* RESPONSIVE DESIGN */

@media (max-width: 1000px) {
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .main-content {
    padding: 20px;
  }
}

@media (max-width: 700px) {
  .portal {
    flex-direction: column;
  }

  .sidebar {
    width: 100%;
    padding: 13px;
  }

  .sidebar-brand {
    padding-bottom: 15px;
  }

  .sidebar-brand img {
    width: 40px;
    height: 40px;
  }

  .menu-label {
    margin-bottom: 8px;
  }

  .nav-link {
    display: inline-flex;
    padding: 10px;
    margin: 2px;
    font-size: 11px;
  }

  .sidebar-bottom {
    padding-top: 10px;
  }

  .logout-button {
    padding: 10px;
  }

  .main-content {
    padding: 13px;
  }

  .topbar {
    padding: 14px;
  }

  .topbar h2 {
    font-size: 16px;
  }

  .user-info strong {
    max-width: 110px;
    overflow-wrap: anywhere;
  }

  .welcome-section {
    align-items: flex-start;
    flex-direction: column;
  }

  .stats-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 10px;
  }

  .stat-card {
    padding: 14px;
  }

  .stat-card h2 {
    font-size: 22px;
  }

  .event-grid,
  .edit-grid {
    grid-template-columns: 1fr;
  }

  .profile-heading {
    flex-wrap: wrap;
  }

  .edit-button {
    margin-left: 0;
  }

  .content-card {
    padding: 17px;
  }
}

@media (max-width: 420px) {
  .auth-card {
    padding: 28px 20px;
  }

  .auth-card h1 {
    font-size: 24px;
  }

  .stats-grid {
    grid-template-columns: 1fr;
  }

  .section-heading {
    align-items: flex-start;
  }
}
</style>
