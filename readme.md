## About

Laravel has become the most popular choice for developing PHP projects. One important reason for this popularity is the built in support for Vue.js, a very fast growing JavaScript library for developing impressive front-ends.

This combination results is fast, secure and very impressive applications that need minimum time to go from ideation to final code review. The support for Vue.js means that Laravel developers could use Vue components easily within their apps without wasting time in writing integrations for the components. To demonstrate the support, I decided to create a single page application in Laravel with a Vue.js powered frontend.

Step by step:
* Download / Install Laravel
  ```shell
  composer create-project --prefer-dist laravel/laravel l5vuespa
  ```
* Download NPM dependencies
  ```shell
  npm install
  ```
* Make migration
  ```shell
  php artisan make:migration create_tasks_table --create=tasks
  ```
  ```php
  public function up()
  {
    Schema::create('tasks', function (Blueprint $table) {
      $table->increments('id');
      $table->string('name');
      $table->unsignedInteger('user_id');
      $table->text('desscription');
      $table->timestamps();
    });
  }
  ```
* Edit **app/Providers/AppServiceProvider.php** to add this code
  ```php
  use Illuminate\Support\Facades\Schema;

  public function boot()
  {
    Schema::defaultStringLength(191);
  }
  ```
* Run Migration
  ```shell
  php artisan migrate
  ```
* Make Authentication Scaffolding
  ```shell
  php make:auth
  ```
* Make Model as well as the Controller
  ```shell
  php artisan make:model Task -r
  ```
* Update **Task** Model with following code for [Mass Assignment](https://laravel.com/docs/5.6/eloquent#mass-assignment)
  ```php
  protected $fillable = [
    'name',
    'user_id',
    'description'
  ];
  ```
* Update **TaskController** CRUD
  * Update `index()` method to get Tasks data and return it as json
    ```php
    use Illuminate\Support\Facades\Auth; 

    public function index()
    {
      $tasks = Task::where(['user_id' => Auth::user()->id])->get();
      return response()->json([
        'tasks' => $tasks,
      ], 200);
    }
    ```
  * Dont forget to use `auth` middleware for protecting page, use the constructor method
    ```php
    public function __constructor()
    {
      $this->middleware('auth')
    }
    ```
  * Update `store()` method for saving data into database
    ```php
    public function store(Request $request)
    {
      $this->validate($request, [
        'name' => 'required|max:255',
        'description' => 'required',
      ]);
      $task = Task::create([
        'name' => request('name'),
        'description' => request('description'),
        'user_id' => Auth::user()->id,
      ]);

      return response()->json([
        'task' => $task,
        'message' => 'Success',
      ], 200);
    }
    ```
  * Update `update()` method for updating data in database
    ```php
    public function update(Request $request, Task $task)
    {
      $this->validate($request, [
        'name' => 'required|max:255',
        'description' => 'required',
      ]);

      $task->name = request('name');
      $task->description = request('description');
      $task->save();

      return response()->json([
        'message' => 'Task updated successfully!'
      ], 200);
    }
    ```
  * Update `destroy()` method for deleting data in database
    ```php
    public function destroy(Task $task)
    {
      $task->delete();

      return response()->json([
        'message' => 'Task deleted successfully!'
      ], 200);
    }
    ```
* Add resource controller to `web.php`
  ```php
  Route::resource('/task', 'TaskController');
  ```
* Create a new file Vue assets saved in **/resource/assets/js/components/Task.vue**
  ```vue
  <template>
    <div class="container">
      <div class="row">
        <div class="col">
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">My Assignments</h5>
              <p class="card-text">
                
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>

  <script>
    export default {
      mounted() {

      }
    }
  </script>
  ```
  * Add following line to register to vue app from **/resources/assets/js/app.js**
    ```js
    Vue.component('task', require('./components/Task.vue'));
    ```
* Compile assets
  ```shell
  npm run dev
  ```
* Calling the vue component from blade, update file **/resources/views/home.blade.php** and match it below
  ```php
  @extends('layouts.app')

  @section('content')
  <task></task>
  @endsection
  ```
* Create, Read, Update & Delete in Task.vue
  This part is too long to read, so just watch it in repo's code, it located in [**/resources/assets/js/components/Task.vue**](./resources/assets/js/components/Task.vue)

Source Tutorial: [Create A Laravel Vue Single Page App In Under An Hour](https://www.cloudways.com/blog/laravel-vue-single-page-app/)