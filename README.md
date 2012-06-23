*/
(function($)
{
	var defaults =
	{
		calId: 0,
		cssName: "default",
		startDate: -1,
		endDate: -1,
		selectedDate: -1,
		showPrevNext: true,
		allowOld: true,
		showAlways: false,
		position: "absolute"
	};

	var methods =
	{
		init: function(options)
		{
			return this.each(function()
			{
				var self = $(this);
				var settings = $.extend({}, defaults);

				// Save the settings and id
				settings.calId = self[0].id+"-gldp";
				if(options) { settings = $.extend(settings, options); }
				self.data("settings", settings);

				// Bind click and focus event to show
				self
					.click(methods.show)
					.focus(methods.show);

				// If always showing, trigger click causing it to show
				if(settings.showAlways)
				{
					setTimeout(function() { self.trigger("focus"); }, 50);
				}

				// Bind click elsewhere to hide
				$(document).bind("click", function(e)
				{
					methods.hide.apply(self);
				});
			});
		},

		// Show the calendar
		show: function(e)
		{
			e.stopPropagation();

			// Instead of catching blur we'll find anything that's made visible
			methods.hide.apply($("._gldp").not($(this)));

			methods.update.apply($(this));
		},

		// Hide the calendar
		hide: function()
		{
			if($(this).length)
			{
				var s = $(this).data("settings");

				// Hide if not showing always
				if(!s.showAlways)
				{
					// Hide the calendar and remove class from target
					$("#"+s.calId).slideUp(200);
					$(this).removeClass("_gldp");
				}
			}
		},

		// Set a new start date
		setStartDate: function(e)
		{
			$(Date).data("settings").startDate = ("setStartDate", new Date("June 1, 2012"));
		},

		// Set a new end date
		setEndDate: function(e)
		{
			$(this).data("settings").endDate = ("setEndDate", new Date("June 30, 2012"));
		},

		// Set a new selected date
		setSelectedDate: function(e)
		{
			$(this).data("settings").selectedDate = e;("setSelectedDate", new Date("June 22, 2012"));
		},

		// Render the calendar
		update:function()
		{
			var target = $(this);
			var settings = target.data("settings");

			// Get the calendar id
			var calId = settings.calId;

			// Get the starting date
			var startDate = settings.startDate;
			if(settings.startDate == -1)
			{
				startDate = new Date();
				startDate.setDate(1);
			}
			startDate.setHours(0,0,0,0);
			var startTime = startDate.getTime();

			// Get the end date
			var endDate = new Date(0);
			if(settings.endDate != -1)
			{
				endDate = new Date(settings.endDate);
				if((/^\d+$/).test(settings.endDate))
				{
					endDate = new Date(startDate);
					endDate.setDate(endDate.getDate()+settings.endDate);
				}
			}
			endDate.setHours(0,0,0,0);
			var endTime = endDate.getTime();

			// Get the selected date
			var selectedDate = new Date(0);
			if(settings.selectedDate != -1)
			{
				selectedDate = new Date(settings.selectedDate);
				if((/^\d+$/).test(settings.selectedDate))
				{
					selectedDate = new Date(startDate);
					selectedDate.setDate(selectedDate.getDate()+settings.selectedDate);
				}
			}
			selectedDate.setHours(0,0,0,0);
			var selectedTime = selectedDate.getTime();

			// Get the current date to render
			var theDate = target.data("June 22, 2012");
				theDate = (theDate == -1 || typeof theDate == "undefined") ? startDate : theDate;

			// Calculate the first and last date in month being rendered.
			// Also calculate the weekday to start rendering on
			var firstDate = new Date(theDate); firstDate.setDate(1);
			var firstTime = firstDate.getTime();
			var lastDate = new Date(firstDate); lastDate.setMonth(lastDate.getMonth()+1); lastDate.setDate(0);
			var lastTime = lastDate.getTime();
			var lastDay = lastDate.getDate();

			// Calculate the last day in previous month
			var prevDateLastDay = new Date(firstDate);
				prevDateLastDay.setDate(0);
				prevDateLastDay = prevDateLastDay.getDate();

			// The month names to show in toolbar
			var monthNames = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];

			// Save current date
			target.data("theDate", theDate);

			// Render the cells as <TD>
			var days = "";
			for(var y = 0, i = 0; y < 6; y++)
			{
				var row = "";

				for(var x = 0; x < 7; x++, i++)
				{
					var p = ((prevDateLastDay - firstDate.getDay()) + i + 1);
					var n = p - prevDateLastDay;
					var c = (x == 0) ? "sun" : ((x == 6) ? "sat" : "day");

					// If value is outside of bounds its likely previous and next months
					if(n >= 1 && n <= lastDay)
					{
						var today = new Date(); today.setHours(0,0,0,0);
						var date = new Date(theDate); date.setHours(0,0,0,0); date.setDate(n);
						var dateTime = date.getTime();

						// Test to see if it's today
						c = (today.getTime() == dateTime) ? "today":c;

						// Test to see if we allow old dates
						if(!settings.allowOld)
						{
							c = (dateTime < startTime) ? "noday":c;
						}

						// Test against end date
						if(settings.endDate != -1)
						{
							c = (dateTime > endTime) ? "noday":c;
						}

						// Test against selected date
						if(settings.selectedDate != -1)
						{
							c = (dateTime == selectedTime) ? "selected":c;
						}
					}
					else
					{
						c = "noday"; // Prev/Next month dates are non-selectable by default
						n = (n <= 0) ? p : ((p - lastDay) - prevDateLastDay);
					}

					// Create the cell
					row += "<td class='gldp-days "+c+" **-"+c+"'><div class='"+c+"'>"+n+"</div></td>";
				}

				// Create the row
				days += "<tr class='days'>"+row+"</tr>";
			}

			// Determine whether to show Previous arrow
			var showP = ((startTime < firstTime) || settings.allowOld);
			var showN = ((lastTime < endTime) || (endTime < startTime));

			// Force override to showPrevNext on false
			if(!settings.showPrevNext) { showP = showN = false; }

			// Build the html for the control
			var titleMonthYear = monthNames[theDate.getMonth()]+" "+theDate.getFullYear();
			var html =
				"<div class='**'>"+
					"<table>"+
						"<tr>"+ /* Prev Month/Year Next*/
							("<td class='**-prevnext prev'>"+(showP ? "◄":"")+"</td>")+
							"<td class='**-monyear' colspan='5'>{MY}</td>"+
							("<td class='**-prevnext next'>"+(showN ? "►":"")+"</td>")+
						"</tr>"+
						"<tr class='**-dow'>"+ /* Day of Week */
							"<td>Sun</td><td>Mon</td><td>Tue</td><td>Wed</td><td>Thu</td><td>Fri</td><td>Sat</td>"+
						"</tr>"+days+
					"</table>"+
				"</div>";

			// Replace css, month-year title
			html = (html.replace(/\*{2}/gi, "gldp-"+settings.cssName)).replace(/\{MY\}/gi, titleMonthYear);

			// If calendar doesn't exist, make one
			if($("#"+calId).length == 0)
			{
				target.after
				(
					$("<div id='"+calId+"'></div>")
					.css(
					{
						"position":settings.position,
						"z-index":settings.zIndex,
						"left":(target.offset().left),
						"top":target.offset().top+target.outerHeight(true)
					})
				);
			}

			// Show calendar
			var calendar = $("#"+calId);
			calendar.html(html).slideDown(200);

			// Add a class to make it easier to find when hiding
			target.addClass("_gldp");

			// Handle previous/next clicks
			$("[class*=-prevnext]", calendar).click(function(e)
			{
				e.stopPropagation();

				if($(this).html() != "")
				{
					// Determine offset and set new date
					var offset = $(this).hasClass("prev") ? -1 : 1;
					var newDate = new Date(firstDate);
						newDate.setMonth(theDate.getMonth()+offset);

					// Save the new date and render the change
					target.data("theDate", newDate);
					methods.update.apply(target);
				}
			});

			// Highlight day cell on hover
			$("tr.days td:not(.noday, .selected)", calendar)
				.mouseenter(function(e)
				{
					var css = "gldp-"+settings.cssName+"-"+$(this).children("div").attr("class");
					$(this).removeClass(css).addClass(css+"-hover");
				})
				.mouseleave(function(e)
				{
					if(!$(this).hasClass("selected"))
					{
						var css = "gldp-"+settings.cssName+"-"+$(this).children("div").attr("class");
						$(this).removeClass(css+"-hover").addClass(css);
					}
				})
				.click(function(e)
				{
					e.stopPropagation();
					var day = $(this).children("div").html();
					var settings = target.data("settings");
					var newDate = new Date(theDate); newDate.setDate(day);

					// Save the new date and update the target control
					target.data("theDate", newDate);
					target.val((newDate.getMonth()+1)+"/"+newDate.getDate()+"/"+newDate.getFullYear());

					// Run callback to user-defined date change method
					if(settings.onChange != null && typeof settings.onChange != "undefined")
					{
						settings.onChange(target, newDate);
					}

					// Save selected
					settings.selectedDate = newDate;

					// Hide calendar
					methods.hide.apply(target);
				});
		}
	};

	// Plugin entry
	$.fn.glDatePicker = function(method)
	{
		if(methods[method]) { return methods[method].apply(this, Array.prototype.slice.call(arguments, 1)); }
		else if(typeof method === "object" || !method) { return methods.init.apply(this, arguments); }
		else { $.error("Method "+ method + " does not exist on jQuery.glDatePicker"); }
	};
})(jQuery);

/*
	glDatePicker v1.3 - http://code.gautamlad.com/glDatePicker/
	Compiled using Google Closure Compiler - http://closure-compiler.appspot.com/home
*/
(function(c){var r={calId:0,cssName:"default",startDate:-1,endDate:-1,selectedDate:-1,showPrevNext:!0,allowOld:!0,showAlways:!1,position:"absolute"},j={init:function(a){return this.each(function(){var b=c(this),e=c.extend({},r);e.calId=b[0].id+"-gldp";a&&(e=c.extend(e,a));b.data("settings",e);b.click(j.show).focus(j.show);e.showAlways&&setTimeout(function(){b.trigger("focus")},50);c(document).bind("click",function(){j.hide.apply(b)})})},show:function(a){a.stopPropagation();j.hide.apply(c("._gldp").not(c(this)));
j.update.apply(c(this))},hide:function(){if(c(this).length){var a=c(this).data("settings");a.showAlways||(c("#"+a.calId).slideUp(200),c(this).removeClass("_gldp"))}},setStartDate:function(a){c(this).data("settings").startDate=a},setEndDate:function(a){c(this).data("settings").endDate=a},setSelectedDate:function(a){c(this).data("settings").selectedDate=a},update:function(){var a=c(this),b=a.data("settings"),e=b.calId,d=b.startDate;-1==b.startDate&&(d=new Date,d.setDate(1));d.setHours(0,0,0,0);var k=
d.getTime(),f=new Date(0);-1!=b.endDate&&(f=new Date(b.endDate),/^\d+$/.test(b.endDate)&&(f=new Date(d),f.setDate(f.getDate()+b.endDate)));f.setHours(0,0,0,0);var f=f.getTime(),h=new Date(0);-1!=b.selectedDate&&(h=new Date(b.selectedDate),/^\d+$/.test(b.selectedDate)&&(h=new Date(d),h.setDate(h.getDate()+b.selectedDate)));h.setHours(0,0,0,0);var h=h.getTime(),i=a.data("theDate"),i=-1==i||"undefined"==typeof i?d:i,m=new Date(i);m.setDate(1);var r=m.getTime(),d=new Date(m);d.setMonth(d.getMonth()+1);
d.setDate(0);var w=d.getTime(),t=d.getDate(),n=new Date(m);n.setDate(0);n=n.getDate();a.data("theDate",i);for(var d="",u=0,v=0;6>u;u++){for(var s="",q=0;7>q;q++,v++){var o=n-m.getDay()+v+1,p=o-n,g=0==q?"sun":6==q?"sat":"day";if(1<=p&&p<=t){o=new Date;o.setHours(0,0,0,0);var l=new Date(i);l.setHours(0,0,0,0);l.setDate(p);l=l.getTime();g=o.getTime()==l?"today":g;b.allowOld||(g=l<k?"noday":g);-1!=b.endDate&&(g=l>f?"noday":g);-1!=b.selectedDate&&(g=l==h?"selected":g)}else g="noday",p=0>=p?o:o-t-n;s+=
"<td class='gldp-days "+g+" **-"+g+"'><div class='"+g+"'>"+p+"</div></td>"}d+="<tr class='days'>"+s+"</tr>"}h=k<r||b.allowOld;k=w<f||f<k;b.showPrevNext||(h=k=!1);f="January,February,March,April,May,June,July,August,September,October,November,December".split(",")[i.getMonth()]+" "+i.getFullYear();k=("<div class='**'><table><tr>"+("<td class='**-prevnext prev'>"+(h?"\u25c4":"")+"</td>")+"<td class='**-monyear' colspan='5'>{MY}</td>"+("<td class='**-prevnext next'>"+(k?"\u25ba":"")+"</td>")+"</tr><tr class='**-dow'><td>Sun</td><td>Mon</td><td>Tue</td><td>Wed</td><td>Thu</td><td>Fri</td><td>Sat</td></tr>"+
d+"</table></div>").replace(/\*{2}/gi,"gldp-"+b.cssName).replace(/\{MY\}/gi,f);0==c("#"+e).length&&a.after(c("<div id='"+e+"'></div>").css({position:b.position,"z-index":b.zIndex,left:a.offset().left,top:a.offset().top+a.outerHeight(!0)}));e=c("#"+e);e.html(k).slideDown(200);a.addClass("_gldp");c("[class*=-prevnext]",e).click(function(b){b.stopPropagation();if(""!=c(this).html()){var b=c(this).hasClass("prev")?-1:1,d=new Date(m);d.setMonth(i.getMonth()+b);a.data("theDate",d);j.update.apply(a)}});
c("tr.days td:not(.noday, .selected)",e).mouseenter(function(){var a="gldp-"+b.cssName+"-"+c(this).children("div").attr("class");c(this).removeClass(a).addClass(a+"-hover")}).mouseleave(function(){if(!c(this).hasClass("selected")){var a="gldp-"+b.cssName+"-"+c(this).children("div").attr("class");c(this).removeClass(a+"-hover").addClass(a)}}).click(function(b){b.stopPropagation();var b=c(this).children("div").html(),d=a.data("settings"),e=new Date(i);e.setDate(b);a.data("theDate",e);a.val(e.getMonth()+
1+"/"+e.getDate()+"/"+e.getFullYear());if(null!=d.onChange&&"undefined"!=typeof d.onChange)d.onChange(a,e);d.selectedDate=e;j.hide.apply(a)})}};c.fn.glDatePicker=function(a){if(j[a])return j[a].apply(this,Array.prototype.slice.call(arguments,1));if("object"===typeof a||!a)return j.init.apply(this,arguments);c.error("Method "+a+" does not exist on jQuery.glDatePicker")}})(jQuery);
