---
title: 請AI協助我開發-part08
date: 2025-09-04 15:19:53
tags:
---

```
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
    Schema::create('exercise_types', function (Blueprint $table) {
        $table->id(); // id INT AUTO_INCREMENT PRIMARY KEY

        $table->string('name', 100); // 運動名稱

        $table->string('intensity', 20)->nullable();

        $table->string('unit', 20);
        // calories_per_unit
        $table->decimal('calories_per_unit', 6, 2);

        $table->text('description')->nullable();

        $table->string('updated_by', 50);

        // created_at, updated_at
        $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('exercise_types');
    }
};
```


