Route 使用规范
===================================== 

根据项目实际，Route采用如下组织形式：
一般用户不需要prefix，直接使用如下‘/’作为根路径。

routes.php

    require_once('admin_routes.php');
    require_once('user_routes.php');
    require_once('wechat_routes.php');

user_routes.php    

    // Route group for namespace of Users\Businesses
    Route::group(['namespace' => 'Users\Businesses'], function() {
        
        // for WelcomeController.
        Route::get('/', 'WelcomeController@index');
        Route::get('/', 'WelcomeController@index');
        Route::get('/', 'WelcomeController@index');

        // for SaleOrderController.
        Route::get('/', 'SaleOrderController@index');

    });


admin使用'admin'作为prefix，访问时使用'/admin/'作为根路径。    

admin_routes.php

    // Route group for prefix of admin.
    Route::group(['prefix' => 'admin'], function() {
        
        // for group for namespace of Admins\Shop.
        Route::group(['namespace' => 'Admins\Shop'], function() {

            // for ShopController.
            Route::get('/', 'ShopController@index');
            Route::get('shop/edit', 'ShopController@edit');
        });
    });
