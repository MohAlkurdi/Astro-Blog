---
title: "Seeding database in Laravel"
description: "How can you seed your database in a Laravel application"
pubDate: "14 Mar 2024"
---

Hey everyone, have ever wondered what are the different ways of seeding a database in Laravel?

## What is seeding exactly?

A [seeder](https://laravel.com/docs/11.x/seeding) is a class used to insert data into the database by running a command. Laravel includes the ability to seed your database with data using seed classes. All seed classes are stored in the **`database/seeders`** directory. By default, a **`DatabaseSeeder`** class is defined for you. From this class, you may use the `call` method to run other seed classes, allowing you to control the seeding order.

## Getting started

So, if you go to your Laravel project under `database` folder, you’ll find inside it a folder called seeders, and inside it, you’ll find a file called `DatabaseSeeder.php` which will have default database seeder class that’s setup for us out of the box, and it contains a `run` method, inside that method, we can add functionality to get data into our database, this how the file will look like:

```php
# database/seeders/DatabaseSeeder.php
class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        // \App\Models\User::factory(10)->create();

        // \App\Models\User::factory()->create([
        //     'name' => 'Test User',
        //     'email' => 'test@example.com',
        // ]);
    }
}
```

## Using models

The simpler way to get data into a database is to just create a new model, yup, it’s that easy! So your file will look like this:

```php
# database/seeders/DatabaseSeeder.php
class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
         \App\Models\User::factory()->create([
             'name' => 'Test User',
             'email' => 'moh@example.com',
             'password' => Hash::make('secret'),
         ]);
    }
}
```

so, to run the seeder, head to your terminal and type:

```bash
php artisan db:seed
```

Now, go check your users table, and you will see a new user created in your table! Congrats, now you've made your first seeder. But you might ask, now what if I want to create multiple users, should I run the command multiple time? Will, you can, but there’s an easier way of doing so!

## Using Factories

Laravel provide to us an easy way to generate these models and add more than one at a time, if you checked under you `database` folder, you'll find another folder called `factories` and a `UserFactory` class provided to us, so if you check the file you'll find a `definition` method which will return back an array of the user model, and we will need that to create a user

```php
# database/factories/UserFactory.php
public function definition(): array
{
	return [
		'name' => fake()->name(),
		'email' => fake()->unique()->safeEmail(),
		'email_verified_at' => now(),
		'password' => static::$password ??= Hash::make('password'),
		'remember_token' => Str::random(10),
	];
}
```

all this attributes are generated using a [Faker library](https://github.com/FakerPHP/Faker), of course you could change it to a hard coded value if you want you. Now that we have learned about all this, how can we use the factory into our seeder?

If you go back to our `DatabaseSeeder.php` file, now we can call the factory method on the `User` model, and we can add how many user instance we want to create!

```php
# database/seeders/DatabaseSeeder.php
public function run(): void
{
	User::factory(10)->create();
}
```

now if you go back to your terminal and run

```bash
php artisan db:seed
```

you'll see 10 users created in you database!

In some cases maybe you created a model and a migration but didn't create the factory, how can you create one? will it's simple, let's say we have a `Post` model and migration, so just go back to your terminal and run:

```bash
php artisan make:factory PostFactory
```

and it'll create a new factory class in `database/factories` directory, and just make sure that you have a `use HasFactory;` in your model class, which mean that you're able use the factory method on the model class, if you've removed it you'll not be able to use factories on your model.

so now go to the factory file that we just created using the `php artisan make` command, and add the attributes that you need, for the `Post` model let's say we want to create a title and a body for our post here's how we can do it:

```php
# database/factories/PostFactory.php
public function definition(): array
{
	return [
		'title' => fake()->title(),
		'body' => fake()->paragraph(),
	];
}
```

so now if you go back to the `DatabaseSedder` class and add:

```php
# database/seeders/DatabaseSeeder.php
public function run(): void
{
	Post::factory(10)->create();
}
```

and run the command:

```bash
php artisan db:seed
```

you'll see 10 post created with a title and a body!

## Seeder Class

Now let's say we want to specify when to run a specific seeder, lucky for us, we can do that !

we just need to hop back to the terminal and run:

```bash
php artisan make:seeder PostSeeder
```

now you'll find a new class under `databse/seeder` folder, and now what you can do is remove the factory call from the `DatabaseSeeder` class and move it to our newly created class

```php
# database/seeders/DatabaseSeeder.php
public function run(): void
{
	Post::factory(10)->create();
}
```

and so on and so fourth for the rest of your seeder, now our `DatabaseSeeder` class is empty. Now if you want to run a seeder for a specific class you'll have to run:

```bash
php artisan db:seed --class=PostSeeder
```

and it'll seed only the post table and nothing more!

A common case you might face is that maybe you want to seed everything again, what you can do is go back to the `DatabaseSeeder` class and add the factory method on the model class, but that might seem like copy and pasting all over again in two different places, instead what we can do is `$this->call()` which takes in an array of seeder classes that are called when the command `db:seed` is used in artisan:

```php
# database/seeders/DatabaseSeeder.php
public function run(): void
{
	$this->call([
		UserSeeder:class,
		PostSeeder:class,
		// add any other seeder you have
	]);
}
```

so this will automatically now to take all the classes in the array and fire up the corresponding `run` method on each class. So now just run in your terminal:

```bash
php artisan db:seed
```

you'll see all your classes are seeded successfully!

---

That's it for this blog post, Thank you for reading!
for additional information, check the [laravel docs](https://laravel.com/docs/11.x/seeding#main-content).
