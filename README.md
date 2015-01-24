# UPME-dropdown-widget
## UPME extension: Dropdown login/register/profile widget

* This is widget for UPME on wordpress. You can buy plugin at: http://codecanyon.net/item/user-profiles-made-easy-wordpress-plugin/4109874
* And I used jquery dropdown that can be found at: http://labs.abeautifulsite.net/jquery-dropdown/

### HOW TO ADD WIDGET?

<b>1. upload files to upme plugin directory:</b>
>	../widgets/upme-login-dropdown-widget.php <br />
>	../css/jquery.dropdown.css <br />
>	../js/jquery.dropdown.js <br />
>	../img/icon-comments.png <br />
>	../img/icon-posts.png <br />
	
<b>2. edit upme.php file</b><br />

	You can also download and overwrite, but be careful and save your source upme.php first!

>	add line 103:
> ```php
>	require_once upme_path . 'widgets/upme-login-dropdown-widget.php';
>	```
>	add line 113: 
> ```php
> wp_register_script('upme_dropdown', upme_url . 'js/jquery.dropdown.js', array('jquery'));
> ```
>	add line 114:
> ```php
>	wp_enqueue_script('upme_dropdown');
> ```
>	add line 115: 
> ```php
>	wp_enqueue_style( 'upme_dropdown_style', upme_url . 'css/jquery.dropdown.css');
> ```
>	add line 116: 
> ```php
>	wp_enqueue_script('upme_dropdown_style');
> ```
	
<b>3. edit classes/class-upme.php</b>

	You can also download and overwrite, but be careful and save your source classes/class-upme.php first!

>	add function upme_dropdown_profile() // you can find it from line 5106 to 5191
> ```php
   function upme_dropdown_profile($widget_settings = array()) {
        global $post;
        /* Capture logged in user ID */
        $current_user = wp_get_current_user();
        if (($current_user instanceof WP_User)) {
            $this->logged_in_user = $current_user->ID;
        }
        $dropdown_class = 'upme-dropdown-widget';
        $name_holder_width = '100%';
        $width = 1;
        // Show custom field as profile title
        $profile_title_field = $this->get_option('profile_title_field');
        // Get value of profile title field or default display name if empty
        $profile_name_display = $this->upme_profile_title_value('first_name', $this->logged_in_user);
		$profile_name_display .= ' ' . $this->upme_profile_title_value('last_name', $this->logged_in_user);
		$profile_nickname_display = $this->upme_profile_title_value('display_name', $this->logged_in_user);
        /* Block profile based on custom status and display information to user */
        $validate_profile_visibility_params = array('user_id' => $this->logged_in_user, 'status' => 'true', 'info' => '', 'context' => 'dropdown_profile');
        $profile_visibility = apply_filters('upme_validate_profile_visibility', $validate_profile_visibility_params);
        if( isset($profile_visibility['status']) && ! $profile_visibility['status'] ){
            $info_display = upme_profile_visibility_info($profile_visibility,$profile_title_display);
            return $info_display;
        }
        /* <-- Block profile --> */
        /* If no ID is set, normally logged out */
        /* User must login to view his profile. */
        /* UPME Filter for customizing profile URL */
        $params = array('id' => $this->logged_in_user, 'view' => null, 'modal' => null, 'group'=> null , 'use_in_sidebar'=> 'yes', 'context'=>'sidebar_widget');
        $profile_url = apply_filters('upme_custom_profile_url',$this->profile_link($this->logged_in_user),$params);
        // End Filter
        /* UPME Filter for customizing profile picture */
        $params = array('id'=> $this->logged_in_user, 'view' => null, 'modal' => null, 'use_in_sidebar'=> 'yes', 'context'=>'sidebar_widget' );
        $profile_pic_display = '<a href="' . $profile_url . '">' . $this->pic($this->logged_in_user, 50) . '</a>';
        $profile_pic_display = apply_filters('upme_custom_profile_pic',$profile_pic_display,$params);
        // End Filter                            
		$display = '';
		$display .= '<span class="dropdown-text" data-dropdown="#dropdown-6">'.$profile_nickname_display.'</span>';
		$display .= '<div id="dropdown-6" class="dropdown">';
		$display .= '<div class="dropdown-panel upme-widget-wrap upme-login upme-sidebar-widget"><div class="upme-inner upme-login-wrapper">';
		$display .= '<div class="dropdown-profile">';
		$display .= '<div class="dropdown-avatar">'.$profile_pic_display.'</div>';
		$display .= '<div class="dropdown-profile-info">';
		$display .= '<span class="profile-name">'.$profile_name_display.'</span>';
		$display .= '<span class="profile-nickname">'.$profile_nickname_display.'</span>';
		$display .= '<ul class="dropdown-links">';
		if ($this->can_edit_profile($this->logged_in_user, $id) == true) {
			$display .= '<li><a href="'.$profile_url.'">'.__('My profile','upme').'</a></li>';
		}
        if (is_user_logged_in ()) {
            //Enable customlogout url
            $logout_url = '';
            if(!empty($widget_settings['logout-link'])){
                $logout_url = ' redirect_to='.$widget_settings['logout-link'];
            }
			$display .= '<li>'.do_shortcode('[upme_logout class="" user_id="' . $this->logged_in_user . '"  '.$logout_url.']').'</li>';
        }
		$display .= '</ul>';
		$display .= '</div>'; // end of div .dropdown-profile-info
		$display .= '</div>'; // end of div .dropdown-profile
		if ($widget_settings['show-stats'] == 1) {
			$display .= '<div class="dropdown-stats">';
			$display .= $this->show_user_stats_dropdown($current_user->ID);
			$display .= '</div>'; // end of div .dropdown-stats
		}
		$display .= '</div></div>';
		$display .= '</div>'; // end of div #dropdown-6
		return $display;
    }
> ```
>	add function show_user_stats_dropdown() // you can find it from line 5193 to 5225
> ```php
	function show_user_stats_dropdown($id, $show_posts_stats = true, $show_comments_stats = true) {
        $author_posts_text = '';
        // Include the link to author posts page based on the setting in admin
        if ($this->get_option('link_author_posts_page') == '1') {
            $author_posts_url = get_author_posts_url($id);
            // Remove link for author who has no post entries
            if (0 == $this->get_entries_num($id)) {
                $author_posts_text = $this->get_entries_num($id);
            } else {
                $author_posts_text = '<a href="' . $author_posts_url . '">' . $this->get_entries_num($id) . '</a>';
            }
        } else {
            $author_posts_text = $this->get_entries_num($id);
        }
        $upme_stats_items = array(
                                'posts' => '<div class="dropdown-stats-i dropdown-stats-posts"><i class="dropdown-icon dropdown-icon-rss"></i><span class="dropdown-posts-link">' . $author_posts_text . '</span></div>',
                                'comments' => '<div class="dropdown-stats-i dropdown-stats-comments"><i class="dropdown-icon dropdown-icon-comments-alt"></i><span class="dropdown-comments-link">' . $this->get_comments_num($id) . '</span></div>',       
                            );
        /* UPME Filter for customizing items in  profile stats section */
        $upme_stats_items = apply_filters('upme_stats_items',$upme_stats_items,$id);
        // End Filter
        $display  = '';
        foreach ($upme_stats_items as $key => $itm) {
            $display .= $itm;
        }
        return $display;
    }	
> ```

You can check how this widget works at: http://www.notranjska.org

I've made it quickly and it can be a little bit messy. I'll be more than happy to get your report on bugs and other stuff. ;-)
