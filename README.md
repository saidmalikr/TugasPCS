# TugasPCS
18.12.0635 - Said Malik Rafsanjani S

// AuthActivity
package com.futuramatata.app.ui.auth

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.databinding.DataBindingUtil
import com.futuramatata.app.R
import com.futuramatata.app.data.model.AuthUser
import com.futuramatata.app.databinding.ActivityAuthBinding
import com.futuramatata.app.ui.home.MainActivity

class AuthActivity : AppCompatActivity() {
    lateinit var binding: ActivityAuthBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_auth)
    }

    fun onSuccess(user: AuthUser?) {
        val intent = Intent(this, MainActivity::class.java)
        startActivity(intent)
        finish()
    }
}

// AuthViewModel
package com.futuramatata.app.ui.auth

import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.futuramatata.app.data.model.ActionState
import com.futuramatata.app.data.model.AuthUser
import com.futuramatata.app.data.repository.AuthRepository
import kotlinx.coroutines.launch

class AuthViewModel(val repo: AuthRepository) : ViewModel() {
    val authUser = repo.authUser
    val isLogin = repo.isLogin
    val authLogin = MutableLiveData<ActionState<AuthUser>>()
    val authRegister = MutableLiveData<ActionState<AuthUser>>()
    val authLogout = MutableLiveData<ActionState<Boolean>>()

    val email = MutableLiveData<String>()
    val password = MutableLiveData<String>()
    val fullName = MutableLiveData<String>()

    fun login() {
        viewModelScope.launch {
            val resp = repo.login(email.value ?: "", password.value ?: "")
            authLogin.value = resp
        }
    }

    fun register() {
        viewModelScope.launch {
            val resp = repo.register(
                AuthUser(
                    email = email.value ?: "",
                    password = password.value ?: "",
                    fullName = fullName.value ?: ""
                )
            )
            authRegister.value = resp
        }
    }

    fun logout() {
        viewModelScope.launch {
            val resp = repo.logout()
            authLogout.value = resp
        }
    }
}

// FuturamatataAuth
package com.futuramatata.app.ui.auth

import android.content.Context
import com.futuramatata.app.data.model.ActionState
import com.futuramatata.app.data.repository.AuthRepository
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext

object FuturamatataAuth {

     fun logout(context: Context, callback: ((ActionState<Boolean>) -> Unit)? = null) {
         val repo = AuthRepository(context)
         CoroutineScope(Dispatchers.IO).launch {
             val resp = repo.logout()
             withContext(Dispatchers.Main) {
                 if (callback != null) callback.invoke(resp)
             }
         }
     }
}

// LoginFragment
package com.futuramatata.app.ui.auth

import android.os.Bundle
import android.util.Log
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Toast
import androidx.navigation.fragment.findNavController
import com.futuramatata.app.R
import com.futuramatata.app.data.repository.AuthRepository
import com.futuramatata.app.databinding.FragmentLoginBinding

class LoginFragment : Fragment() {
    val parent: AuthActivity by lazy { activity as AuthActivity }
    val viewModel: AuthViewModel by lazy { AuthViewModel(AuthRepository(parent)) }
    lateinit var binding: FragmentLoginBinding

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentLoginBinding.inflate(inflater, container, false)
        binding.lifecycleOwner = this
        binding.viewModel = viewModel
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        init()
        observe()
    }

    private fun init() {
        binding.buttonRegister.setOnClickListener {
            findNavController().navigate(LoginFragmentDirections.actionLoginFragmentToRegisterFragment())
        }
    }

    private fun observe() {
        viewModel.authLogin.observe(viewLifecycleOwner) {
            if (it.isConsumed) {
                Log.i("Login", "isConsumed")
            } else if (!it.isSuccess) {
                Toast.makeText(parent, it.message, Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(parent, it.message, Toast.LENGTH_SHORT).show()
                parent.onSuccess(it.data)
            }
            it.isConsumed = true
        }
    }


}

// RegisterFragment
package com.futuramatata.app.ui.auth

import android.os.Bundle
import android.util.Log
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Toast
import androidx.navigation.fragment.findNavController
import com.futuramatata.app.R
import com.futuramatata.app.data.repository.AuthRepository
import com.futuramatata.app.databinding.FragmentRegisterBinding

class RegisterFragment : Fragment() {
    val parent: AuthActivity by lazy { activity as AuthActivity }
    val viewModel: AuthViewModel by lazy { AuthViewModel(AuthRepository(parent)) }
    lateinit var binding: FragmentRegisterBinding

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentRegisterBinding.inflate(inflater, container, false)
        binding.lifecycleOwner = this
        binding.viewModel = viewModel
        return binding.root

    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        init()
        observe()
    }

    private fun init() {
        binding.buttonLogin.setOnClickListener {
            findNavController().navigate(RegisterFragmentDirections.actionRegisterFragmentToLoginFragment())
        }
    }

    private fun observe() {
        viewModel.authRegister.observe(viewLifecycleOwner) {
            if (it.isConsumed) {
                Log.i("Register", "isConsumed")
            } else if (!it.isSuccess) {
                Toast.makeText(parent, it.message, Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(parent, it.message, Toast.LENGTH_SHORT).show()
                findNavController().navigate(RegisterFragmentDirections.actionRegisterFragmentToLoginFragment())
            }
            it.isConsumed = true
        }
    }


}

// SplashFragment
package com.futuramatata.app.ui.auth

import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.navigation.fragment.findNavController
import com.futuramatata.app.R
import com.futuramatata.app.data.repository.AuthRepository
import com.futuramatata.app.databinding.FragmentSplashBinding
import kotlinx.coroutines.*

class SplashFragment : Fragment() {
    val parent: AuthActivity by lazy { activity as AuthActivity }
    val viewModel: AuthViewModel by lazy { AuthViewModel(AuthRepository(parent)) }
    lateinit var binding: FragmentSplashBinding

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = FragmentSplashBinding.inflate(inflater, container, false)
        binding.lifecycleOwner = this
        binding.viewModel = viewModel
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        init()
    }

    private fun init() {
        CoroutineScope(Dispatchers.IO).launch {
            delay(2000)
            withContext(Dispatchers.Main) {
                if (viewModel.isLogin.value == true) {
                    parent.onSuccess(viewModel.authUser.value)
                } else {
                    findNavController().navigate(SplashFragmentDirections.actionSplashFragmentToLoginFragment())
                }
            }
        }
    }

}

// fragment_login
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="viewModel"
            type="com.futuramatata.app.ui.auth.AuthViewModel" />
    </data>

    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".ui.auth.LoginFragment">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <ImageView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:src="@drawable/bg_auth"/>

            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:gravity="center"
                android:orientation="vertical"
                android:padding="16dp"
                app:layout_constraintWidth_percent="0.75"
                app:layout_constraintVertical_bias="0.85"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toTopOf="parent">

                <com.google.android.material.textfield.TextInputLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
                    android:hint="@string/email">

                    <com.google.android.material.textfield.TextInputEditText
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:id="@+id/editEmail"
                        android:inputType="textEmailAddress"
                        android:text="@={viewModel.email}"/>

                </com.google.android.material.textfield.TextInputLayout>

                <com.google.android.material.textfield.TextInputLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
                    android:layout_marginTop="8dp"
                    android:hint="@string/password">

                    <com.google.android.material.textfield.TextInputEditText
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:id="@+id/editPassword"
                        android:inputType="textPassword"
                        android:text="@={viewModel.password}"/>

                </com.google.android.material.textfield.TextInputLayout>

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/buttonLogin"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:insetBottom="0dp"
                    android:insetTop="0dp"
                    android:onClick="@{()-> viewModel.login()}"
                    android:text="@string/login"
                    android:layout_marginTop="8dp"/>

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/buttonRegister"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:insetBottom="0dp"
                    android:insetTop="0dp"
                    android:text="@string/register"
                    android:layout_marginTop="32dp"/>
            </LinearLayout>

        </androidx.constraintlayout.widget.ConstraintLayout>

    </androidx.coordinatorlayout.widget.CoordinatorLayout>
</layout>

// fragment_register
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="viewModel"
            type="com.futuramatata.app.ui.auth.AuthViewModel" />
    </data>

    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".ui.auth.LoginFragment">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <ImageView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:src="@drawable/bg_auth"/>

            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:gravity="center"
                android:orientation="vertical"
                android:padding="16dp"
                app:layout_constraintWidth_percent="0.75"
                app:layout_constraintVertical_bias="0.85"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toTopOf="parent">

                <com.google.android.material.textfield.TextInputLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
                    android:hint="@string/email">

                    <com.google.android.material.textfield.TextInputEditText
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:id="@+id/editEmail"
                        android:inputType="textEmailAddress"
                        android:text="@={viewModel.email}"/>

                </com.google.android.material.textfield.TextInputLayout>

                <com.google.android.material.textfield.TextInputLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
                    android:layout_marginTop="8dp"
                    android:hint="@string/password">

                    <com.google.android.material.textfield.TextInputEditText
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:id="@+id/editPassword"
                        android:inputType="textPassword"
                        android:text="@={viewModel.password}"/>

                </com.google.android.material.textfield.TextInputLayout>

                <com.google.android.material.textfield.TextInputLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
                    android:layout_marginTop="8dp"
                    android:hint="@string/full_name">

                    <com.google.android.material.textfield.TextInputEditText
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:id="@+id/editFullName"
                        android:inputType="textCapWords"
                        android:text="@={viewModel.fullName}"/>

                </com.google.android.material.textfield.TextInputLayout>

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/buttonRegister"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:insetBottom="0dp"
                    android:insetTop="0dp"
                    android:onClick="@{()-> viewModel.register()}"
                    android:text="@string/register"
                    android:layout_marginTop="8dp"/>

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/buttonLogin"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:insetBottom="0dp"
                    android:insetTop="0dp"
                    android:text="@string/login"
                    android:layout_marginTop="32dp"/>
            </LinearLayout>

        </androidx.constraintlayout.widget.ConstraintLayout>

    </androidx.coordinatorlayout.widget.CoordinatorLayout>
</layout>

// fragment_splash
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="viewModel"
            type="com.futuramatata.app.ui.auth.AuthViewModel" />
    </data>

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".ui.auth.SplashFragment">

    </FrameLayout>
</layout>
