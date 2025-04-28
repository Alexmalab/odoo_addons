# Odoo Module: mass_mailing_themes

Category: Marketing/Email Marketing

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Mass Mailing Themes',
    'summary': 'Design gorgeous mails',
    'description': """
Design gorgeous mails
    """,
    'version': '1.2',
    'sequence': 110,
    'website': 'https://www.odoo.com/app/mailing',
    'category': 'Marketing/Email Marketing',
    'depends': [
        'mass_mailing',
    ],
    'data': [
        'data/ir_attachment_data.xml',
        'views/mass_mailing_themes_templates.xml'
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\ir_attachment_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mass_mailing_themes.s_tech_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_tech_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing_themes/static/src/img/theme_blogging/s_tech_default_image.jpg</field>
        </record>
    </data>
    <data>
        <record id="mass_mailing_themes.s_default_image_block_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_default_image_block_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing_themes/static/src/img/theme_training/s_default_image_block_image.jpg</field>
        </record>
    </data>
</odoo>

```

## File: static\src\img\theme_training\see_you_soon.svg

```svg
<svg width="1044" height="554" viewBox="0 0 1044 554" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M378.646 402.384H369.966V414.094H378.646V402.384Z" fill="#F9B499"/>
<path d="M424.646 395.364L427.346 406.584H437.456L432.066 393.784L424.646 395.364Z" fill="#F9B499"/>
<path d="M348.716 422.534C348.716 421.166 349.26 419.853 350.228 418.885C351.195 417.918 352.508 417.374 353.876 417.374H360.616C361.59 417.276 362.52 416.921 363.311 416.344C364.102 415.768 364.725 414.991 365.116 414.094C365.116 414.094 367.806 407.714 370.956 409.954C370.956 409.954 372.956 410.404 372.306 413.104H378.426C379.079 413.1 379.718 413.287 380.266 413.643C380.813 413.998 381.244 414.506 381.506 415.104C382.36 417.468 382.671 419.994 382.416 422.494H374.306L373.226 419.374L372.556 422.514L348.716 422.534Z" fill="#2A4F97"/>
<path d="M426.896 417.374C426.389 416.362 426.124 415.246 426.124 414.114C426.124 412.982 426.389 411.866 426.896 410.854C426.896 410.854 425.326 401.424 433.746 404.004V405.464L435.806 405.344C436.3 405.311 436.793 405.427 437.221 405.676C437.648 405.926 437.992 406.298 438.206 406.744C438.596 407.544 439.076 408.564 439.496 409.534C439.78 410.252 439.777 411.052 439.486 411.767C439.196 412.482 438.64 413.058 437.936 413.374C435.926 414.244 432.476 415.584 426.896 417.374Z" fill="#2A4F97"/>
<path d="M660.376 408.674L665.226 424.054C665.226 424.054 671.796 429.114 674.496 424.054L672.796 407.874L660.376 408.674Z" fill="#FFBF9D"/>
<path d="M692.326 405.534L699.476 422.534L708.406 422.874L704.386 399.384L692.326 405.534Z" fill="#FFBF9D"/>
<path d="M522.206 420.184H512.436V433.494H522.206V420.184Z" fill="#F9B499"/>
<path d="M568.006 418.944L569.526 433.494L578.656 432.594L577.106 417.374L568.006 418.944Z" fill="#F9B499"/>
<path d="M510.756 442.594V435.764C510.756 434.759 511.156 433.795 511.866 433.084C512.577 432.373 513.541 431.974 514.546 431.974H515.636C515.636 431.974 519.686 425.574 523.386 431.974C523.386 431.974 523.926 435.614 525.726 437.704C526.183 438.235 526.431 438.914 526.426 439.614V439.614C526.426 440.41 526.11 441.173 525.548 441.735C524.985 442.298 524.222 442.614 523.426 442.614L510.756 442.594Z" fill="#F09E47"/>
<path d="M565.536 442.594C565.536 442.594 565.356 430.124 571.416 432.024C572.116 432.426 572.915 432.62 573.72 432.585C574.526 432.55 575.305 432.286 575.966 431.824C575.966 431.824 580.136 424.824 582.226 429.934C582.226 429.934 585.636 436.194 591.896 437.144L598.796 437.404C599.64 437.435 600.439 437.792 601.024 438.4C601.61 439.008 601.937 439.82 601.936 440.664V442.594H576.346L574.786 439.224V442.594H565.536Z" fill="#F09E47"/>
<path d="M529.796 134.294V123.494C529.796 123.494 522.046 121.144 529.796 116.544C529.796 116.544 543.956 108.114 556.026 112.044C556.026 112.044 559.566 114.744 557.026 121.264C557.026 121.264 561.306 126.204 555.576 134.854L554.896 127.104L540.896 125.974L537.896 139.124L533.516 137.664L529.796 134.294Z" fill="#2A4F97"/>
<path d="M532.866 139.534C532.866 139.534 530.226 153.014 523.226 158.614L532.226 162.434L544.996 160.644L543.156 148.724C543.156 148.724 554.886 155.024 555.586 134.014C556.193 131.186 556.193 128.262 555.586 125.434C555.586 125.434 552.286 127.554 549.586 123.584C549.586 123.584 540.116 122.834 539.796 125.754C539.796 125.754 537.676 142.754 535.876 137.214C536.208 135.976 536.303 134.687 536.156 133.414C536.063 132.698 535.761 132.026 535.287 131.481C534.813 130.936 534.19 130.542 533.494 130.35C532.798 130.158 532.061 130.176 531.374 130.4C530.688 130.625 530.084 131.047 529.636 131.614C529.537 131.751 529.447 131.895 529.366 132.044C529.366 132.044 527.226 135.224 532.866 139.534Z" fill="#F9B499"/>
<path d="M360.956 263.484C360.956 263.484 363.956 322.114 367.696 335.254C367.696 335.254 366.346 387.824 369.376 404.004H379.826C379.826 404.004 379.156 395.244 382.526 381.084C382.526 381.084 392.916 349.424 385.866 333.744L389.316 324.804C389.316 324.804 392.406 337.304 393.196 340.154L423.976 398.614L433.746 395.364C433.746 395.364 427.346 352.364 409.146 328.454C409.146 328.454 418.246 292.144 411.506 263.454H359.226" fill="#F09E47"/>
<path d="M356.576 179.004C356.576 179.004 373.646 145.624 403.756 163.654C403.756 163.654 426.226 174.734 429.816 201.914H406.446C406.446 201.914 415.666 240.914 415.886 266.384C415.888 266.621 415.842 266.856 415.752 267.076C415.662 267.296 415.53 267.495 415.363 267.663C415.195 267.832 414.996 267.965 414.777 268.056C414.558 268.147 414.323 268.194 414.086 268.194H360.906C360.665 268.194 360.426 268.146 360.203 268.052C359.981 267.957 359.78 267.819 359.612 267.646C359.444 267.472 359.313 267.267 359.226 267.041C359.139 266.816 359.098 266.575 359.106 266.334L361.516 188.884L356.576 179.004Z" fill="#2A4F97"/>
<path d="M362.636 178.714H369.226C370.232 178.716 371.21 179.045 372.012 179.651C372.815 180.257 373.399 181.107 373.676 182.074L389.226 236.754C389.349 237.198 389.368 237.664 389.282 238.116C389.196 238.568 389.006 238.995 388.729 239.362C388.451 239.73 388.093 240.028 387.682 240.235C387.27 240.442 386.817 240.551 386.356 240.554H383.566L362.636 178.714Z" fill="#FAFAFA"/>
<path d="M487.666 203.154H514.666C514.666 203.154 504.226 219.754 510.226 238.244C510.226 238.244 510.906 249.754 509.226 258.294C509.226 258.294 505.516 265.674 512.226 265.244H559.406C559.406 265.244 566.146 266.934 562.776 258.294C562.776 258.294 552.666 241.904 565.136 203.154H575.926C575.488 193.15 572.299 183.461 566.71 175.153C561.12 166.845 553.347 160.24 544.246 156.064C544.246 156.064 544.356 162.594 533.576 161.064C533.576 161.064 525.706 161.174 524.576 157.244C524.626 157.204 500.816 161.254 487.666 203.154Z" fill="#465C68"/>
<path d="M491.636 203.154C491.636 203.154 491.006 206.204 490.366 210.664C488.056 226.794 508.776 235.374 518.536 222.374C522.006 217.724 525.916 212.174 530.246 205.544C530.246 205.544 534.956 202.954 536.756 191.784V178.104C536.756 178.104 539.456 171.364 534.756 173.604C534.756 173.604 524.196 181.924 524.866 200.004L510.226 212.374C510.114 212.46 509.98 212.512 509.84 212.526C509.699 212.539 509.558 212.513 509.431 212.451C509.304 212.389 509.197 212.293 509.122 212.173C509.047 212.053 509.007 211.915 509.006 211.774L513.176 203.184L491.636 203.154Z" fill="#F9B499"/>
<path d="M508.426 263.484C508.426 263.484 499.126 289.764 511.936 347.724C511.936 347.724 504.486 373.334 508.546 392.884L511.936 422.884H522.716C522.716 422.884 532.826 357.504 533.496 348.074L537.496 321.784L547.966 346.054C547.966 346.054 542.236 356.834 554.706 384.464C554.706 384.464 565.486 412.774 566.836 422.884H578.626C578.626 422.884 572.566 366.604 569.196 349.084C569.196 349.084 567.076 276.614 563.586 263.494L508.426 263.484Z" fill="#2A4F97"/>
<path d="M574.596 203.154L574.956 208.924C574.956 208.924 589.446 194.924 592.476 192.534C592.476 192.534 589.946 184.114 592.476 180.534C592.476 180.534 598.376 173.794 601.406 175.814C600.517 177.244 599.441 178.549 598.206 179.694L595.206 184.744V188.374L603.286 188.204L604.636 185.844C604.636 185.844 604.766 178.784 609.376 177.844C610.113 177.701 610.877 177.845 611.511 178.246C612.145 178.648 612.601 179.277 612.786 180.004C613.256 181.864 613.246 184.764 610.366 187.844C610.366 187.844 605.146 194.084 600.926 194.414C600.926 194.414 588.466 221.044 571.106 231.994C571.106 231.994 566.526 238.764 559.946 225.284C560.516 221.554 561.286 217.534 562.326 213.284C563.206 209.664 564.186 206.284 565.206 203.184C568.327 202.861 571.473 202.851 574.596 203.154V203.154Z" fill="#F9B499"/>
<path d="M600.546 165.354L594.076 189.504H607.026L612.986 165.354H600.546Z" fill="#2A4F97"/>
<path d="M629.706 235.704C629.706 235.704 627.706 223.064 623.616 213.534C623.616 213.534 624.866 213.064 625.806 210.724C625.806 210.724 627.676 203.854 625.496 200.894C625.402 200.679 625.348 200.449 625.336 200.214C625.337 199.529 625.069 198.87 624.591 198.379C624.113 197.887 623.462 197.602 622.776 197.584C621.922 197.613 621.087 197.842 620.338 198.254C619.589 198.666 618.948 199.248 618.466 199.954C618.466 199.954 614.876 202.614 617.376 214.324C617.376 214.324 617.056 243.824 621.746 250.854C621.746 250.854 624.556 256.944 631.266 252.574C631.266 252.574 642.666 240.864 649.846 226.964L648.446 208.964L641.576 205.774C641.576 205.774 631.896 229.774 629.706 235.704Z" fill="#F9B499"/>
<path d="M690.556 214.704C690.556 214.704 698.466 264.144 696.946 288.914C696.946 288.914 691.886 295.994 693.156 302.914C693.156 302.914 694.416 304.914 695.256 299.204C695.256 299.204 697.116 297.694 697.116 300.044C696.793 302.583 696.223 305.085 695.416 307.514C695.308 307.792 695.259 308.089 695.27 308.386C695.282 308.684 695.355 308.976 695.484 309.244C695.614 309.513 695.797 309.751 696.023 309.946C696.248 310.14 696.512 310.286 696.796 310.374C698.546 310.904 700.946 310.594 702.506 306.114C702.506 306.114 704.866 302.404 703.856 289.114C703.856 289.114 708.856 230.024 705.036 212.734L695.616 213.734L690.556 214.704Z" fill="#F9B499"/>
<path d="M642.946 255.804C642.946 255.804 621.456 302.564 660.886 410.074H672.886C672.886 410.074 672.716 362.214 664.626 345.364L666.986 332.934C666.986 332.934 682.826 384.454 693.606 408.384L705.066 404.384C705.066 404.384 700.006 361.924 691.586 343.224C691.586 343.224 710.776 288.494 690.336 258.164L642.946 255.804Z" fill="#2A4F97"/>
<path d="M649.426 441.744C649.428 440.878 649.664 440.028 650.109 439.285C650.555 438.542 651.193 437.934 651.956 437.524L660.396 432.994C662.014 432.12 663.244 430.671 663.846 428.934L665.746 423.424C665.801 423.278 665.908 423.159 666.048 423.09C666.187 423.022 666.348 423.009 666.496 423.054L670.046 424.224C670.359 424.328 670.693 424.35 671.017 424.289C671.34 424.227 671.643 424.085 671.896 423.874L673.066 422.874C673.254 422.72 673.475 422.61 673.711 422.553C673.948 422.496 674.194 422.493 674.432 422.545C674.67 422.597 674.893 422.702 675.084 422.853C675.275 423.003 675.43 423.195 675.536 423.414C676.409 425.235 676.873 427.225 676.896 429.244V440.534C676.896 440.847 676.772 441.147 676.551 441.368C676.329 441.59 676.029 441.714 675.716 441.714V441.714C675.403 441.714 675.103 441.59 674.882 441.368C674.661 441.147 674.536 440.847 674.536 440.534V431.694L669.716 439.454C669.282 440.147 668.679 440.717 667.963 441.113C667.248 441.508 666.444 441.715 665.626 441.714L649.426 441.744Z" fill="#F09E47"/>
<path d="M687.006 438.544C687.006 438.544 684.816 442.254 689.196 441.744L703.006 439.744C703.612 439.653 704.18 439.391 704.642 438.989C705.104 438.587 705.442 438.062 705.616 437.474L708.406 428.114L710.546 437.344C710.609 437.605 710.757 437.838 710.967 438.004C711.178 438.171 711.438 438.263 711.706 438.264V438.264C711.868 438.263 712.027 438.229 712.175 438.164C712.323 438.1 712.456 438.005 712.567 437.888C712.677 437.77 712.763 437.631 712.817 437.479C712.872 437.327 712.896 437.165 712.886 437.004C712.656 433.414 711.806 423.184 709.236 419.334C709.148 419.203 709.032 419.091 708.897 419.008C708.762 418.925 708.611 418.872 708.454 418.852C708.297 418.832 708.137 418.846 707.986 418.893C707.834 418.94 707.695 419.019 707.576 419.124L706.496 420.124C706.04 420.535 705.498 420.837 704.91 421.01C704.321 421.184 703.702 421.222 703.096 421.124L699.746 420.574C699.596 420.549 699.441 420.555 699.293 420.592C699.145 420.63 699.006 420.698 698.886 420.792C698.765 420.886 698.666 421.004 698.594 421.139C698.521 421.274 698.478 421.422 698.466 421.574L698.096 428.444C698.049 429.238 697.821 430.01 697.431 430.703C697.042 431.396 696.5 431.991 695.846 432.444L687.006 438.544Z" fill="#F09E47"/>
<path d="M342.226 217.304L332.126 182.804C331.988 182.328 331.963 181.826 332.052 181.338C332.142 180.85 332.343 180.39 332.641 179.993C332.938 179.596 333.324 179.274 333.768 179.052C334.211 178.831 334.7 178.715 335.196 178.714H366.776C367.47 178.716 368.145 178.943 368.699 179.361C369.253 179.78 369.655 180.367 369.846 181.034L385.706 236.484C385.843 236.959 385.867 237.46 385.777 237.946C385.686 238.432 385.484 238.89 385.187 239.285C384.889 239.68 384.504 240 384.061 240.22C383.618 240.44 383.131 240.555 382.636 240.554H351.846C351.165 240.555 350.502 240.338 349.953 239.935C349.404 239.532 348.999 238.964 348.796 238.314L342.226 217.304Z" fill="#2A4F97"/>
<path d="M428.726 201.914H406.446L409.626 220.424C409.663 220.65 409.652 220.882 409.592 221.104C409.532 221.325 409.426 221.531 409.28 221.708C409.134 221.885 408.953 222.029 408.747 222.13C408.541 222.231 408.316 222.287 408.086 222.294L390.386 222.694C386.796 211.804 383.386 212.254 383.386 212.254C381.586 211.914 382.386 217.864 382.386 217.864H378.226C370.966 211.964 370.806 214.724 370.806 214.724C369.126 216.514 376.586 220.074 376.586 220.074C376.298 220.463 376.107 220.915 376.031 221.393C375.955 221.871 375.996 222.361 376.151 222.819C376.305 223.278 376.568 223.693 376.917 224.028C377.266 224.363 377.691 224.609 378.156 224.744C376.156 228.464 379.156 229.324 379.156 229.324C377.916 235.614 389.316 232.024 389.316 232.024L413.606 236.864C415.402 237.227 417.268 236.919 418.852 235.997C420.435 235.076 421.625 233.605 422.196 231.864C422.196 231.864 427.946 208.204 428.726 201.914Z" fill="#F9B499"/>
<path d="M379.156 153.614V159.314C379.156 159.314 387.746 169.164 396.336 160.014C396.336 160.014 391.016 156.014 389.126 143.434H384.886L379.156 153.614Z" fill="#F9B499"/>
<path d="M665.496 179.004C665.496 179.004 650.376 183.554 640.686 207.944L647.516 210.724C647.516 210.724 651.116 226.854 648.866 237.864C648.866 237.864 643.476 254.484 641.676 259.434H691.106C688.244 254.842 685.614 250.109 683.226 245.254C681.709 241.986 681.114 238.366 681.506 234.784C682.339 227.737 685.031 221.038 689.306 215.374L705.036 212.684C705.036 212.684 703.036 186.174 680.766 177.414C680.766 177.414 678.266 182.414 670.766 181.994C670.226 181.994 669.696 181.934 669.156 181.914C668.016 181.914 665.666 181.544 665.496 179.004Z" fill="#F09E47"/>
<path d="M666.136 181.764C665.957 181.675 665.798 181.549 665.67 181.394C665.542 181.24 665.448 181.06 665.393 180.867C665.339 180.674 665.325 180.472 665.353 180.274C665.381 180.075 665.45 179.884 665.556 179.714C668.556 174.714 667.676 168.564 667.676 168.564C666.355 168.349 665.097 167.844 663.994 167.085C662.89 166.326 661.969 165.332 661.296 164.174C660.6 163.041 660.128 161.785 659.906 160.474C658.967 155.343 659.31 150.06 660.906 145.094C660.906 145.094 667.196 136.894 673.826 146.214C673.826 146.214 674.616 156.444 677.986 156.104C677.986 156.104 681.686 152.284 683.826 154.754C683.826 154.754 686.096 157.364 680.696 162.194L677.756 164.074C677.756 164.074 679.956 174.804 680.206 180.074C680.226 180.409 680.13 180.74 679.936 181.013C679.741 181.285 679.459 181.484 679.136 181.574L675.516 182.574C672.396 183.459 669.059 183.171 666.136 181.764V181.764Z" fill="#F9B499"/>
<path d="M661.346 145.464C661.346 145.464 655.716 141.254 660.216 133.284C660.216 133.284 664.486 125.194 674.476 130.924C676.383 132.866 678.079 135.005 679.536 137.304C680.038 137.844 680.667 138.248 681.366 138.481C682.065 138.713 682.812 138.766 683.536 138.634C684.254 138.564 684.977 138.594 685.686 138.724C687.307 139.025 688.816 139.763 690.049 140.857C691.282 141.951 692.194 143.361 692.686 144.934C693.214 146.486 693.418 148.13 693.286 149.764C693.061 152.867 692.264 155.901 690.936 158.714C689.965 160.709 689.408 162.879 689.296 165.094C689.186 167.614 689.656 170.534 691.876 172.374C691.876 172.374 703.566 183.494 700.306 190.904C700.306 190.904 692.666 198.324 683.006 194.274C680.804 190.983 679.886 186.997 680.426 183.074C679.928 176.69 679.026 170.344 677.726 164.074C677.726 164.074 684.226 158.354 684.126 155.874C684.126 155.874 684.486 153.674 681.276 154.014C680.004 154.403 678.858 155.125 677.956 156.104C677.956 156.104 674.696 157.104 673.796 146.214C673.806 146.194 667.226 137.264 661.346 145.464Z" fill="#2A4F97"/>
<path d="M379.156 157.864V155.794C380.621 155.318 381.972 154.546 383.127 153.526C384.281 152.505 385.213 151.259 385.866 149.864C385.75 151.74 385.035 153.529 383.828 154.968C382.62 156.408 380.983 157.423 379.156 157.864V157.864Z" fill="#F4A48E"/>
<path d="M365.446 135.974C365.446 135.974 364.066 149.754 372.636 154.864C374.205 155.801 376.033 156.21 377.851 156.03C379.67 155.85 381.382 155.091 382.736 153.864C385.179 151.43 386.997 148.442 388.036 145.154C388.036 145.154 393.756 138.004 387.356 134.414H379.356L371.266 130.814L365.446 135.974Z" fill="#F9B499"/>
<path d="M359.446 132.904C359.272 132.94 359.092 132.928 358.924 132.871C358.756 132.813 358.606 132.712 358.49 132.578C358.374 132.443 358.296 132.281 358.263 132.106C358.231 131.931 358.246 131.751 358.306 131.584C359.306 129.124 361.606 125.464 366.546 125.354H371.226C371.226 125.354 379.336 119.514 385.506 123.734C385.506 123.734 390.506 125.354 388.746 134.594C388.746 134.594 392.806 136.054 391.826 144.334C391.826 144.334 394.746 149.334 389.396 147.414L387.866 144.894C387.719 144.643 387.674 144.345 387.741 144.062C387.808 143.779 387.982 143.532 388.226 143.374C389.106 142.834 390.146 141.484 389.786 138.034C389.741 137.383 389.503 136.761 389.102 136.246C388.701 135.731 388.156 135.347 387.536 135.144C387.176 135.045 386.799 135.021 386.429 135.074C386.059 135.128 385.704 135.257 385.386 135.454C384.9 135.766 384.495 136.189 384.205 136.688C383.915 137.187 383.747 137.748 383.716 138.324C383.616 139.534 383.716 141.104 384.856 141.634C384.856 141.634 383.166 143.124 381.186 139.374C380.659 138.377 380.047 137.426 379.356 136.534C377.636 134.364 374.856 131.674 373.506 135.074C373.506 135.074 372.386 137.944 370.806 135.074C370.806 135.074 371.146 132.194 369.016 134.214C369.016 134.214 364.636 141.214 363.626 136.124C363.626 136.124 360.626 139.124 360.256 135.124C360.256 135.124 360.256 133.484 361.256 132.974C360.665 132.824 360.048 132.8 359.446 132.904Z" fill="#2A4F97"/>
<path d="M543.126 148.724L543.536 151.354C541.921 150.979 540.467 150.098 539.388 148.839C538.308 147.579 537.66 146.008 537.536 144.354C537.536 144.354 540.106 148.194 543.126 148.724Z" fill="#F4A48E"/>
<path d="M667.626 168.544C667.696 169.383 667.696 170.226 667.626 171.064C670.252 170.123 672.479 168.313 673.936 165.934C673.936 165.934 670.366 169.374 667.626 168.544Z" fill="#F4A48E"/>
</svg>

```

## File: views\mass_mailing_themes_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Add themes in editor -->
    <template id="email_designer_snippets" inherit_id="mass_mailing.email_designer_snippets">
        <xpath expr="//div[@id='email_designer_themes']" position="inside">
            <div data-name="event" title="Event Promo" data-img="/mass_mailing_themes/static/src/img/theme_imgs/event_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_event_template"/>
            </div>
            <div data-name="newsletter" title="Newsletter" data-layout-styles="background-color: rgb(255, 255, 255) !important;" data-img="/mass_mailing_themes/static/src/img/theme_imgs/newsletter_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_newsletter_template"/>
            </div>
            <div data-name="training" title="Training" data-layout-styles="background-color: rgb(220, 234, 255) !important;" data-img="/mass_mailing_themes/static/src/img/theme_imgs/training_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_training_template"/>
            </div>
            <div data-name="coupon" title="Coupon Code" data-img="/mass_mailing_themes/static/src/img/theme_imgs/coupon_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_coupon_template"/>
            </div>
            <div data-name="coffeebreak" title="Coffee break" data-layout-styles="background-color: rgb(238, 233, 226) !important;" data-img="/mass_mailing_themes/static/src/img/theme_imgs/coffee_break_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_coffeebreak_template"/>
            </div>
            <div data-name="blogging" title="Blogging" data-img="/mass_mailing_themes/static/src/img/theme_imgs/blogging_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_blogging_template"/>
            </div>
            <div data-name="magazine" title="Magazine" data-img="/mass_mailing_themes/static/src/img/theme_imgs/magazine_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_magazine_template"/>
            </div>
            <div data-name="bignews" title="Big News" data-img="/mass_mailing_themes/static/src/img/theme_imgs/bignews_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_bignews_template"/>
            </div>
            <div data-name="promotion" title="Promotion Program" data-img="/mass_mailing_themes/static/src/img/theme_imgs/promotion_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_promotion_template"/>
            </div>
            <div data-name="roadshow1" title="Roadshow Schedule" data-layout-styles="background-color: rgb(255, 255, 255) !important;" data-img="/mass_mailing_themes/static/src/img/theme_imgs/roadshow_schedule_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_roadshow1_template"/>
            </div>
            <div data-name="roadshow2" title="Roadshow Follow-up" data-layout-styles="background-color: rgb(255, 255, 255) !important;" data-img="/mass_mailing_themes/static/src/img/theme_imgs/roadshow_followup_thumb" data-images-info='{"logo": {"format": "png"}, "all": {"module": "mass_mailing_themes"}}'>
                <t t-call="mass_mailing_themes.theme_roadshow2_template"/>
            </div>
        </xpath>
    </template>

    <!-- Theme "Event" default template -->
    <template id="theme_event_template">
        <style id="design-element">
            h3 {
                color: #2b3b58;
                font-size: 18px;
                font-weight: bolder;
            }
        </style>
        <t t-set="now" t-value="datetime.datetime.now()"/>
        <div class="s_header_social o_mail_block_header_social o_mail_snippet_general pt16 pb16" data-snippet="s_mail_block_header_social" style="background-color: rgb(26, 36, 54) !important;" data-name="Header Social">
            <div class="container">
                <div class="row">
                    <div class="col-lg-4">
                        <a style="text-decoration:none;float:none;" href="http://www.example.com" target="_blank">
                             <img border="0" src="/mass_mailing_themes/static/src/img/theme_event/s_default_image_logo.png" style="height: auto; max-width: 100%; width: 194px;"  width="194" alt="Your Logo" class="img-fluid" />
                        </a>
                    </div>
                    <div class="col-lg-8 o_mail_header_social" style="text-align: right;" t-translation="off">
                        <a title="Facebook" href="https://www.facebook.com/Odoo">
                            <span class="fa fa-facebook text-white" style="background-color: rgb(43, 59, 88) !important;"/>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" title="LinkedIn" href="https://www.linkedin.com/company/odoo">
                            <span class="fa fa-linkedin text-white" style="background-color: rgb(43, 59, 88) !important;"/>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" title="Twitter" href="https://twitter.com/Odoo">
                            <span class="fa fa-twitter text-white" style="background-color: rgb(43, 59, 88) !important;"/>​
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" title="Instagram" href="https://www.instagram.com/explore/tags/odoo/">
                            <span class="fa fa-instagram text-white" style="background-color: rgb(43, 59, 88) !important;"/>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" title="TikTok" href="https://www.tiktok.com/@odoo">
                            <span class="fa fa-tiktok text-white" style="background-color: rgb(43, 59, 88) !important;"/>
                        </a>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general pb16 pt32" style="background-color: rgb(43, 59, 88) !important; padding-left: 15px; padding-right: 15px;" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p><font style="color: rgb(255, 255, 255);">Hi there!</font></p>
                <p><font style="color: rgb(255, 255, 255);">The next edition of <span style="font-weight: bolder;">Odoo Experience Online</span> is coming soon!
                    <br/>To stir up your curiosity, have a look at all the great talks scheduled and highlighted in our agenda!</font>
                </p>
            </div>
        </div>
        <div class="s_media_list o_mail_snippet_general o_cc o_cc2 pb16 pt0" style="background-color: rgb(43, 59, 88) !important; padding-left: 15px; padding-right: 15px;" data-snippet="s_media_list" data-vcss="001" data-name="Media List">
            <div class="container">
                <div class="row s_nb_column_fixed">
                    <div class="col-lg-12 s_media_list_item pt0 pb0" data-name="Media item">
                        <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                            <div class="col-lg-5 s_media_list_img_wrapper align-self-stretch px-0">
                                <img src="/mass_mailing_themes/static/src/img/theme_event/s_default_image_media_list_1.jpg" class="s_media_list_img h-100 w-100" style="padding: 10px;"/>
                            </div>
                            <div class="col-lg-7 s_media_list_body">
                                <div class="s_title o_mail_snippet_general pb0 pt0" data-snippet="s_title" data-name="Title">
                                    <div class="container s_allow_columns">
                                        <h3>Get 6-months' worth of knowledge<br/>in just 2 days</h3>
                                    </div>
                                </div>
                                <p>Learn through workshops, product demos, and inspiring talks from Odoo Experts and Partners.</p>
                                <a class="btn btn-primary" href="http://">See what we've got planned</a>
                            </div>
                        </div>
                    </div>
                    <div class="col-lg-12 s_media_list_item pt16 pb0" data-name="Media item">
                        <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                            <div class="col-lg-5 s_media_list_img_wrapper align-self-stretch px-0">
                                <img src="/mass_mailing_themes/static/src/img/theme_event/s_default_image_media_list_2.jpg" class="s_media_list_img h-100 w-100" style="padding: 10px;"/>
                            </div>
                            <div class="col-lg-7 s_media_list_body">
                                <div class="s_title o_mail_snippet_general pb0 pt0" data-snippet="s_title" data-name="Title">
                                    <div class="container s_allow_columns">
                                        <h3>Meet (other) awesome members<br/>of the community</h3>
                                    </div>
                                </div>
                                <p>Create your very own chat room or join an existing one that sparks your interest.</p>
                                <a class="btn btn-primary" href="http://">Go to the chatrooms</a>
                            </div>
                        </div>
                    </div>
                    <div class="col-lg-12 s_media_list_item pt16 pb0" data-name="Media item">
                        <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                            <div class=" col-lg-5 s_media_list_img_wrapper align-self-stretch px-0">
                                <img src="/mass_mailing_themes/static/src/img/theme_event/s_default_image_media_list_3.jpg" class="s_media_list_img h-100 w-100" style="padding: 10px;"/>
                            </div>
                            <div class="col-lg-7 s_media_list_body">
                                <div class="s_title o_mail_snippet_general pb0 pt0" data-snippet="s_title" data-name="Title">
                                    <div class="container s_allow_columns">
                                        <h3>An interactive, fun<br/>and comprehensive experience</h3>
                                    </div>
                                </div>
                                <p>Experience everything from a virtual exhibitor's hall to multiple interactive presentations and demos.</p>
                                <a class="btn btn-primary" href="http://">Find out more</a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_call_to_action o_mail_snippet_general o_cc o_cc3 pt40 pb40" style="background-color: rgb(26, 36, 54) !important;" data-snippet="s_call_to_action" data-name="Call to Action">
            <div class="container">
                <div class="row">
                    <div class="col-lg-12" style="border-color: rgb(56, 62, 69) !important;">
                        <h3 style="text-align: center;"><font style="color: rgb(255, 255, 255);">And it won't cost you a thing!</font></h3>
                        <p style="text-align: center;"><font style="color: rgb(255, 255, 255);">Zero, zilch, nada! Odoo Experience is free for everyone!</font></p>
                    </div>
                    <div class="col-lg-12" style="text-align: center;">
                        <a class="btn btn-primary btn-lg" href="http://">Register now!</a>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general bg-200 pt16 pb16" data-snippet="s_mail_block_footer_social" data-name="Footer Center">
            <div class="container">
                <div class="row">
                    <div class="col-lg-12 o_mail_footer_links pt8 pb8" style="text-align: center;">
                        <a role="button" href="/unsubscribe_from_list" class="btn btn-link ">Unsubscribe</a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-12">
                        <p style="text-align: center;">
                            © <t t-out="now.strftime('%Y')" /> All Rights Reserved
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Theme "Blogging" default template -->
    <template id="theme_blogging_template">
        <style id="design-element">
            h1 {
                font-weight: bolder;
                color: #36a6d2;
                font-family: Courier New, Courier, "Lucida Sans Typewriter", "Lucida Typewriter", monospace;
            }
            h2 {
                font-weight: bolder;
                font-family: Courier New, Courier, "Lucida Sans Typewriter", "Lucida Typewriter", monospace;
            }
            h3 {
                font-size: 18px;
                font-weight: bolder;
                color: #36a6d2;
                font-family: Courier New, Courier, "Lucida Sans Typewriter", "Lucida Typewriter", monospace;
            }
            p, p > *, li, li > * {
                color: #adadad;
            }
            a:not(.btn), a.btn.btn-link {
                color: #36a6d2;
            }
            a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary {
                background-color: #36a6d2;
                border-color: #36a6d2;
            }
            hr {
                border-top-color: #e9ecef !important;
            }
        </style>
        <t t-set="now" t-value="datetime.datetime.now()"/>
        <div class="s_header_social o_mail_block_header_social o_mail_snippet_general pt40 pb32" data-snippet="s_mail_block_header_social" data-name="Header Social">
            <div class="container">
                <div class="row">
                    <div class="col-lg-4">
                        <a t-att-href="(company_id.website) or '#'" style="text-decoration:none;float:none;" target="_blank">
                            <img border="0" width="189" style="height:auto; max-width:100%; width: 189px;" src="/mass_mailing_themes/static/src/img/theme_blogging/tech_logo.png" class="img-fluid"/>
                        </a>
                    </div>
                    <div class="col-lg-8 o_mail_header_social" style="text-align:right;">
                        <a title="Facebook" href="https://www.facebook.com/Odoo">
                            <span class="fa fa-facebook" style="color: rgb(54, 166, 210) !important; font-size: 18px;"/>​
                        </a>
                        &amp;nbsp; &amp;nbsp;
                        <a style="margin-left:10px" title="LinkedIn" href="https://www.linkedin.com/company/odoo">
                            <span class="fa fa-linkedin" style="color: rgb(54, 166, 210) !important; font-size: 18px;"/>​
                        </a>
                        &amp;nbsp; &amp;nbsp;
                        <a style="margin-left:10px" title="Twitter" href="https://twitter.com/Odoo">
                            <span class="fa fa-twitter" style="color: rgb(54, 166, 210) !important; font-size: 18px;"/>​
                        </a>
                        &amp;nbsp; &amp;nbsp;
                        <a style="margin-left:10px" title="Instagram" href="https://www.instagram.com/explore/tags/odoo/">
                            <span class="fa fa-instagram" style="color: rgb(54, 166, 210) !important; font-size: 18px;"/>
                        </a>
                        &amp;nbsp; &amp;nbsp;
                        <a style="margin-left:10px" title="TikTok" href="https://www.tiktok.com/@odoo">
                            <span class="fa fa-tiktok" style="color: rgb(54, 166, 210) !important; font-size: 18px;"/>
                        </a>
                        &amp;nbsp; &amp;nbsp;
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt0 pb0" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_text_image o_mail_snippet_general" data-snippet="s_text_image" data-name="Text - Image">
            <div class="container">
                <div class="row align-items-center">
                    <div class="col-lg-7 pt16 pb16">
                        <h1>Breaking IT news
                            <br/>and Analysis</h1>
                        <p style="text-align: left;">Our roundup of the most popular and latest tech related videos from last week.<br/></p>
                        <p style="text-align:left;"><a class="btn btn-primary" href="#">VISIT OUR WEBSITE</a></p>
                    </div>
                    <div class="col-lg-5 px-0">
                        <img class="img-fluid o_we_custom_image" src="/web_editor/image_shape/mass_mailing_themes.s_tech_default_image/web_editor/geometric/geo_cornered_triangle.svg" data-shape="web_editor/geometric/geo_cornered_triangle" data-file-name="text_image-cornered_triangle.svg" data-shape-colors=";;;;" data-original-mimetype="image/jpeg"/>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt0 pb16" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_media_list o_mail_snippet_general o_cc o_cc2 pb0 pt0 bg-white" style="padding-left: 15px; padding-right: 15px;" data-snippet="s_media_list" data-vcss="001" data-name="Media List">
            <div class="container">
                <div class="row s_nb_column_fixed s_col_no_bgcolor">
                    <div class="col-lg-12 s_media_list_item pb0 pt0" data-name="Media item">
                        <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                            <div class="col-lg-4 s_media_list_img_wrapper align-self-stretch px-0">
                                <img src="/mass_mailing_themes/static/src/img/theme_blogging/s_default_image_media_list_1.jpg" class="s_media_list_img h-100 w-100"/>
                            </div>
                            <div class="col-lg-8 s_media_list_body">
                                <h3>The future of smartphone video production</h3>
                                <p>Modern smartphones have all the built-in tech equal to a majority of the current professional cameras.</p>
                                <a class="btn btn-primary" href="http://"><span class="fa fa-play" />&amp;nbsp;&amp;nbsp;WATCH VIDEO</a>
                                &amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;
                                <a class="btn btn-link" href="#">READ MORE</a>
                            </div>
                        </div>
                    </div>
                    <div class="col-lg-12 s_media_list_item pt0 pb0" data-name="Media item">
                        <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                            <div class="col-lg-12 s_media_list_img_wrapper align-self-stretch px-0">
                                <div class="s_hr o_mail_snippet_general pt16 pb16" data-snippet="s_hr" data-name="Separator">
                                    <hr class="s_hr_1px s_hr_solid"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-lg-12 s_media_list_item pb0 pt0" data-name="Media item">
                        <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                            <div class="col-lg-4 s_media_list_img_wrapper align-self-stretch px-0">
                                <img src="/mass_mailing_themes/static/src/img/theme_blogging/s_default_image_media_list_2.jpg" class="s_media_list_img h-100 w-100"/>
                            </div>
                            <div class="col-lg-8 s_media_list_body pt16 pb16">
                                <h3>5 Ways Drones Can Better Your Live Events</h3>
                                <p>One clever use of drones is for delivering gifts — but another that's becoming popular for event safety purposes.</p>
                                <a class="btn btn-primary" href="http://">READ MORE</a>
                            </div>
                        </div>
                    </div>
                    <div class="col-lg-12 s_media_list_item pt0 pb0" data-name="Media item">
                        <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                            <div class="col-lg-12 s_media_list_img_wrapper align-self-stretch px-0">
                                <div class="s_hr o_mail_snippet_general pt16 pb16" data-snippet="s_hr" data-name="Separator">
                                    <hr class="s_hr_1px s_hr_solid"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-lg-12 s_media_list_item pt0 pb0" data-name="Media item">
                        <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                            <div class="col-lg-4 s_media_list_img_wrapper align-self-stretch px-0">
                                <img src="/mass_mailing_themes/static/src/img/theme_blogging/s_default_image_media_list_3.jpg" class="s_media_list_img h-100 w-100"/>
                            </div>
                            <div class="col-lg-8 s_media_list_body">
                                <h3>How to Code a Website Without Experience</h3>
                                <p>Anyone with an ounce of business acumen knows that any business that wants to succeed needs a website.</p>
                                <a class="btn btn-primary" href="http://">READ MORE</a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt16 pb0" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general pt16 pb16" data-snippet="s_mail_block_footer_social" data-name="Footer Social Center">
            <div class="container">
                <div class="row">
                    <div class="col-lg-12 o_mail_footer_links pt8 pb8" style="text-align: center;">
                        <a href="/unsubscribe_from_list" class="btn btn-link"><font style="color: rgb(173, 173, 173);">Unsubscribe</font></a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-12">
                        <p style="text-align: center;">
                            © <t t-out="now.strftime('%Y')" /> All Rights Reserved
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Theme "Coupon" default template -->
    <template id="theme_coupon_template">
        <style id="design-element">
            h1 {
                font-size: 48px;
                font-weight: bolder;
                font-family: Georgia, Times, "Times New Roman", serif;
            }
            h2 {
                font-family: Georgia, Times, "Times New Roman", serif;
            }
            h3 {
                font-size: 18px;
                font-weight: bolder;
                font-family: Georgia, Times, "Times New Roman", serif;
            }
            p, p > *, li, li > * {
                font-family: Georgia, Times, "Times New Roman", serif;
            }
            a:not(.btn), a.btn.btn-link {
                color: #C69678;
            }
            a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary {
                background-color: #C69678;
                border-color: #C69678;
            }
            a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary {
                background-color: #fadcc0;
                color: #242530;
                border-color: #c69678;
            }
            hr {
                border-top-color: #c69678 !important;
            }
        </style>
        <t t-set="now" t-value="datetime.datetime.now()"/>
        <t t-set="future_date" t-value="(datetime.date.today() + datetime.timedelta(days=30))"/>
        <div class="s_header_logo o_mail_block_header_logo o_mail_snippet_general" data-snippet="s_mail_block_header_logo" data-name="Centered Logo" style="background-color: rgb(36, 37, 48) !important;">
            <div class="container">
                <div class="row">
                    <div class="col-lg-4" style="text-align: center;">
                        <p><font style="color: rgb(255, 255, 255);">FREE</font>
                        <br/><span style="font-weight: bolder"><font style="color: rgb(255, 255, 255);">SHIPPING</font></span></p>
                    </div>
                    <div class="col-lg-4 pt16 pb16" style="text-align: center;">
                        <a style="text-decoration:none; text-align:center;" t-att-href="(company_id.website) or '#'" target="_blank">
                            <img border="0" src="/mass_mailing_themes/static/src/img/theme_coupon/vip_logo.png" style="height: auto; max-width: 100%; width: 105px;" width="105" class="img-fluid"/>
                        </a>
                    </div>
                    <div class="col-lg-4" style="text-align: center;">
                        <p><font style="color: rgb(255, 255, 255);">EXCLUSIVE</font>
                        <br/><span style="font-weight: bolder"><font style="color: rgb(255, 255, 255);">DEALS</font></span></p>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_cover o_mail_snippet_general" data-snippet="s_cover" data-name="Cover" style="background-color: rgb(36, 37, 48) !important;">
            <img src="/mass_mailing_themes/static/src/img/theme_coupon/vip_banner_part1.png" alt="Cover" class="img-fluid w-100 mx-auto"/>
        </div>
        <div class="s_hr o_mail_snippet_general pt0 pb16" data-snippet="s_hr" data-name="Separator" style="background-color: rgb(36, 37, 48) !important;">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_title o_mail_snippet_general pb8 pt8" data-snippet="s_title" data-name="Title" style="background-color: rgb(36, 37, 48) !important;">
            <div class="container s_allow_columns">
                <h1 style="text-align:center; color: white;">$100 OFF</h1>
                <h2 style="text-align:center; color: white;">VIP members only</h2>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt16 pb0" data-snippet="s_hr" data-name="Separator" style="background-color: rgb(36, 37, 48) !important;">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_cover o_mail_snippet_general" data-snippet="s_cover" data-name="Cover" style="background-color: rgb(36, 37, 48) !important;">
            <img src="/mass_mailing_themes/static/src/img/theme_coupon/vip_banner_part2.png" alt="Cover" class="img-fluid w-100 mx-auto"/>
        </div>
        <div class="s_discount2 o_mail_block_discount2 o_mail_snippet_general pt32 pb32" data-snippet="s_coupon_code" data-name="Promo Code" style="background-color: rgb(36, 37, 48) !important; text-align: center; padding-left: 15px; padding-right: 15px;">
            <p style="text-align: center; color: white;">Here's your coupon code*:</p>
            <table border="0" cellpadding="0" cellspacing="0" align="center" class="border" style="background-color: rgb(36, 37, 48) !important; border-collapse: collapse; border-color: rgb(198, 150, 120) !important;">
                <tr>
                    <td width="50" height="50" align="center" class="o_mail_no_resize" style="min-width: 50px; max-width: 5.6rem; width: 50px !important; background-color: rgb(198, 150, 120) !important; text-align: center;"><i class="fa fa-2x fa-ticket" style="color: rgb(36, 37, 48) !important;"/>​</td>
                    <td width="200" height="50" align="center" style="min-width: 150px; width: 200px;"><p class="mb0"><span style="font-weight: bolder; color: white;">VIP10</span></p></td>
                </tr>
            </table>
            <p style="text-align: center;">
                <font style="color: rgb(113, 114, 127);">*But hurry! Offer ends on <t t-out="future_date.strftime('%m/%d/%Y')"/></font><br/>
            </p>
            <p style="text-align:center;">
                <a class="btn btn-primary btn-lg" href="http://">Redeem it now</a>
            </p>
        </div>
        <div class="s_call_to_action o_mail_snippet_general o_cc o_cc3 pt24 pb24" data-snippet="s_call_to_action" data-name="Call to Action" style="background-color: rgb(36, 37, 48) !important;">
            <div class="container">
                <div class="row">
                    <div class="col-lg-10 offset-lg-1 border pt24 pb16" style="border-width: 2px !important; border-color: rgb(198, 150, 120) !important;">
                        <h3 style="text-align: center;">Thank you for being one of our most loyal customers.</h3>
                        <p style="text-align: center;">Join the community to see our latest news, events, discounts and much more!</p>
                        <p style="text-align: center;">
                            <a class="btn btn-primary" href="http://">Join us</a>
                        </p>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general pt16 pb16" data-snippet="s_mail_block_footer_social" data-name="Footer Center" style="background-color: rgb(36, 37, 48) !important;">
            <div class="container">
                <div class="row">
                    <div class="col-lg-12" style="text-align: center;" t-translation="off">
                        <a aria-label="Facebook" title="Facebook" href="https://www.facebook.com/Odoo">
                            <span class="fa fa-facebook" style="background-color: rgb(198, 150, 120) !important; color: rgb(36, 37, 48) !important;"></span>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="LinkedIn" title="LinkedIn" href="https://www.linkedin.com/company/odoo">
                            <span class="fa fa-linkedin" style="background-color: rgb(198, 150, 120) !important; color: rgb(36, 37, 48) !important;"></span>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="Twitter" title="Twitter" href="https://twitter.com/Odoo">
                            <span class="fa fa-twitter" style="background-color: rgb(198, 150, 120) !important; color: rgb(36, 37, 48) !important;"></span>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="Instagram" title="Instagram" href="https://www.instagram.com/explore/tags/odoo/">
                            <span class="fa fa-instagram" style="background-color: rgb(198, 150, 120) !important; color: rgb(36, 37, 48) !important;"></span>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="TikTok" title="TikTok" href="https://www.tiktok.com/@odoo">
                            <span class="fa fa-tiktok" style="background-color: rgb(198, 150, 120) !important; color: rgb(36, 37, 48) !important;"></span>
                        </a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-12 o_mail_footer_links pt8 pb8" style="text-align: center;">
                        <a role="button" href="/unsubscribe_from_list" class="btn btn-link">Unsubscribe</a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-12">
                        <p style="text-align: center;">
                            © <t t-out="now.strftime('%Y')" /> All Rights Reserved
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Theme "Magazine" default template -->
    <template id="theme_magazine_template">
        <style id="design-element">
            h1 {
                font-weight: bolder;
                font-family: Tahoma, Verdana, Segoe, sans-serif;
            }
            h2 {
                font-size: 18px;
                font-weight: bolder;
                font-family: Tahoma, Verdana, Segoe, sans-serif;
            }
            h3 {
                font-size: 14px;
                font-weight: bolder;
                color: #ffffff;
                font-family: Tahoma, Verdana, Segoe, sans-serif;
            }
            p, p > *, li, li > * {
                font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
            }
            a:not(.btn), a.btn.btn-link {
                color: #6c757d;
            }
            hr {
                border-top-color: #ced4da !important;
            }
        </style>
        <t t-set="now" t-value="datetime.datetime.now()"/>
        <div class="s_header_logo o_mail_block_header_logo o_mail_snippet_general bg-white pt32 pb16" data-snippet="s_mail_block_header_logo" data-name="Centered Logo">
            <div class="container">
                <div class="row">
                    <div class="col-lg-3">
                        <p style="text-align: center;"><t t-out="now.strftime('%b')" />
                        <br/><span style="font-weight: bolder;"><t t-out="now.strftime('%Y')" /></span></p>
                    </div>
                    <div class="col-lg-6">
                        <h1 style="text-align:center;">
                            THE DAILY
                        </h1>
                    </div>
                    <div class="col-lg-3">
                        <p style="text-align: center;">Issue
                            <br/><span style="font-weight: bolder;">#42</span>
                        </p>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pb32" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_text_image o_mail_snippet_general" data-snippet="s_text_image" data-name="Image - Text">
            <div class="container">
                <div class="row align-items-center">
                    <div class="col-lg-8 px-0">
                        <img src="/mass_mailing_themes/static/src/img/theme_magazine/s_default_image_image_text.jpg" class="img w-100"/>
                    </div>
                    <div class="col-lg-4 o_cc pt0 pb0">
                        <h2 style="text-align: center;">IN THIS ISSUE</h2>
                        <div class="s_hr o_mail_snippet_general pt16 pb16" data-snippet="s_hr" data-name="Separator">
                            <hr class="s_hr_1px s_hr_solid"/>
                        </div>
                        <p>
                            <a class="btn btn-link" style="text-align:left" href="#">
                                Interview with Janet James&amp;nbsp;
                                <span class="fa fa-angle-right"/>
                            </a>
                            <br/>
                            <a class="btn btn-link" style="text-align:left" href="#">
                                Destinations for an eco-holiday&amp;nbsp;
                                <span class="fa fa-angle-right"/>
                            </a>
                            <br/>
                            <a class="btn btn-link" style="text-align:left" href="#">
                                Top 5 energy-boosting morning routines&amp;nbsp;
                                <span class="fa fa-angle-right"/>
                            </a>
                            <br/>
                            <a class="btn btn-link" style="text-align:left" href="#">
                                Delicious recipes from around the world&amp;nbsp;
                                <span class="fa fa-angle-right"/>
                            </a>
                            <br/>
                            <a class="btn btn-link" style="text-align:left" href="#">
                                Rainy day ideas&amp;nbsp;
                                <span class="fa fa-angle-right"/>
                            </a>
                        </p>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt32 pb32" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_mail_product_list o_mail_snippet_general pt0" data-snippet="s_product_list" data-name="Items">
            <div class="container">
                <div class="row">
                    <div class="col-lg-6 o_cc">
                        <h3><font style="background-color: rgb(231, 148, 57);">TRAVEL</font></h3>
                        <a href="#">
                            <img class="img-fluid o_we_custom_image" src="/mass_mailing_themes/static/src/img/theme_magazine/s_default_image_product_2.jpg" alt="Man on a rock looking at mountains in the distance" style="width: 100% !important; padding: 15px 0px;"/>
                        </a>
                        <h2 style="text-align: left;">Destinations for an eco-holiday</h2>
                        <p style="text-align: left;">Bitten by the travel bug but looking to travel consciously? Eco-travel is the way forward.</p>
                        <p style="text-align: right;">
                            <a href="#" class="btn btn-link">
                                Find out more&amp;nbsp;<span class="fa fa-angle-right"/>
                            </a>
                        </p>
                    </div>
                    <div class="col-lg-6 o_cc">
                        <h3><font style="background-color: rgb(231, 99, 99);">LIFESTYLE</font></h3>
                        <a href="#">
                            <img class="img-fluid o_we_custom_image" src="/mass_mailing_themes/static/src/img/theme_magazine/s_default_image_product_1.jpg" alt="Woman having breakfast in bed" style="width: 100% !important; padding: 15px 0px;"/>
                        </a>
                        <h2 style="text-align: left;">Top 5 energy-boosting morning routines</h2>
                        <p style="text-align: left;">Imagine waking up with more energy, more flexibility, with less stress and fewer instances of anxiously rushing out the door.</p>
                        <p style="text-align: right;">
                            <a href="#" class="btn btn-link">
                                Find out more&amp;nbsp;<span class="fa fa-angle-right"/>
                            </a>
                        </p>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt16 pb32" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_text_image o_mail_snippet_general" data-snippet="s_text_image" data-name="Text - Image">
            <div class="container">
                <div class="row align-items-center">
                    <div class="col-lg-6 o_cc pt16 pb16">
                        <h3 style="text-align: left;"><font style="background-color: rgb(57, 123, 33);">FOOD</font></h3>
                        <h2>Delicious recipes from around the world</h2>
                        <p>Travel without leaving your kitchen with these international recipes. From Canada to Australia, Mexico to Sweden, and everywhere in between.</p>
                        <div style="text-align:right">
                            <a href="#" class="btn btn-link">
                                Read More&amp;nbsp;<span class="fa fa-angle-right"/>
                            </a>
                        </div>
                    </div>
                    <div class="col-lg-6 px-0">
                        <img src="/mass_mailing_themes/static/src/img/theme_magazine/s_default_image_image_text_2.jpg" class="img w-100"/>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt32 pb16" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general pt16 pb16" data-snippet="s_mail_block_footer_social" data-name="Footer Center">
            <div class="container">
                <div class="row">
                    <div class="col-lg-12" style="text-align: center;" t-translation="off">
                        <a aria-label="Facebook" title="Facebook" href="https://www.facebook.com/Odoo">
                            <span class="fa fa-facebook"></span>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="LinkedIn" title="LinkedIn" href="https://www.linkedin.com/company/odoo">
                            <span class="fa fa-linkedin"></span>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="Twitter" title="Twitter" href="https://twitter.com/Odoo">
                            <span class="fa fa-twitter"></span>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="Instagram" title="Instagram" href="https://www.instagram.com/explore/tags/odoo/">
                            <span class="fa fa-instagram"></span>
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="TikTok" title="TikTok" href="https://www.tiktok.com/@odoo">
                            <span class="fa fa-tiktok"></span>
                        </a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-12 o_mail_footer_links pt8 pb8" style="text-align: center;">
                        <a role="button" href="/unsubscribe_from_list" class="btn btn-link">Unsubscribe</a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-12">
                        <p style="text-align: center;">
                            © <t t-out="now.strftime('%Y')" /> All Rights Reserved
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Theme "Big News" default template -->
    <template id="theme_bignews_template">
        <style id="design-element">
            h1 {
                font-size: 36px;
                font-weight: bolder;
                font-style: italic;
                color: #BA0013;
            }
            h2 {
                font-size: 14px;
                font-weight: bolder;
            }
            h3 {
                font-size: 18px;
                font-weight: bolder;
            }
            p, p > *, li, li > * {
                color: #6c757d;
            }
            a:not(.btn), a.btn.btn-link{
                color: #BA0013;
            }
            a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary {
                background-color: #BA0013;
                border-color: #BA0013;
            }
        </style>
        <t t-set="now" t-value="datetime.datetime.now()"/>
        <t t-set="future_date" t-value="(datetime.date.today() + datetime.timedelta(days=30))"/>
        <div class="s_header_logo o_mail_block_header_logo o_mail_snippet_general pt16 pb16 bg-200" data-snippet="s_mail_block_header_logo" data-name="Header Logo">
            <div class="container">
                <div class="row">
                    <div class="col-lg-4"></div>
                    <div class="col-lg-4 pt16 pb16" style="text-align: center;">
                        <a style="text-decoration:none;" href="http://www.example.com">
                            <img src="/mass_mailing_themes/static/src/img/theme_bignews/bignews_logo.png" width="70" style="height:auto;max-width:100%; width:70px;" class="img-fluid"/>
                        </a>
                    </div>
                    <div class="col-lg-4" style="text-align:right"></div>
                </div>
            </div>
        </div>
        <div class="s_title o_mail_snippet_general pt32 pb16" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <h1 style="text-align: center;">WE'RE MOVING</h1>
            </div>
        </div>
        <div class="s_cover o_mail_snippet_general pt0" data-snippet="s_cover" data-name="Cover">
            <img src="/mass_mailing_themes/static/src/img/theme_bignews/s_default_image_cover.jpg" alt="Cover image" class="img-fluid w-100 mx-auto" style="padding: 15px;"/>
        </div>
        <div class="s_text_block o_mail_snippet_general pt32 pb32" data-snippet="s_text_block" data-name="Text" style="padding-left: 15px; padding-right: 15px;">
            <div class="container s_allow_columns">
                <h2>BEGINNING <t t-out="future_date.strftime('%B %d').upper()" /></h2>
                <p>We are proud to announce that due to our remarkable growth in the Springfield area, we are moving to a new location on July 1.
                   <br/>We will continue to offer the same friendly service at our new address on <span style="font-weight: bolder">1600 Main Street</span>,
                   <br/>which will allow us to offer an even larger selection of products and services.
                </p>
                <p>See you there,</p>
                <p><img src="/mass_mailing_themes/static/src/img/theme_bignews/signature.png" style="width:125px; margin-top:8px;margin-bottom:-25px;" alt="Signature" class="img-fluid"/></p>
                <p>Michael Fletcher<br/>
                   <span style="font-size: 12px; font-weight: bolder;">Customer Service</span>
                </p>
                <p style="text-align: center;">
                    <a role="button" href="#" class="btn btn-primary">READ MORE</a>
                </p>
            </div>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general bg-200 pt16 pb16" data-snippet="s_mail_block_footer_social" data-name="Footer Center">
            <div class="container">
                <div class="row">
                    <div class="col-lg-12" style="text-align: center;" t-translation="off">
                        <a aria-label="Facebook" title="Facebook" href="https://www.facebook.com/Odoo">
                            <span class="fa fa-facebook" style="color: rgb(186, 0, 19) !important;"/>​
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="LinkedIn" title="LinkedIn" href="https://www.linkedin.com/company/odoo">
                            <span class="fa fa-linkedin" style="color: rgb(186, 0, 19) !important;"/>​
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="Twitter" title="Twitter" href="https://twitter.com/Odoo">
                            <span class="fa fa-twitter" style="color: rgb(186, 0, 19) !important;"/>​
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="Instagram" title="Instagram" href="https://www.instagram.com/explore/tags/odoo/">
                            <span class="fa fa-instagram" style="color: rgb(186, 0, 19) !important;"/>​
                        </a>&amp;nbsp;&amp;nbsp;
                        <a style="margin-left:10px" aria-label="TikTok" title="TikTok" href="https://www.tiktok.com/@odoo">
                            <span class="fa fa-tiktok" style="color: rgb(186, 0, 19) !important;"/>​
                        </a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-12 o_mail_footer_links pt8 pb8" style="text-align: center;">
                        <a role="button" href="/unsubscribe_from_list" class="btn btn-link">Unsubscribe</a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-12">
                        <p style="text-align: center;">
                            © <t t-out="now.strftime('%Y')" /> All Rights Reserved
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Theme "Promotion" default template -->
    <template id="theme_promotion_template">
        <style id="design-element">
            h1 {
                font-weight: bolder;
            }
            h2 {
                font-weight: bolder;
            }
            h3 {
                font-size: 18px;
                font-weight: bolder;
            }
            a:not(.btn), a.btn.btn-link {
                color: #000000;
            }
            a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary {
                background-color: #000000;
                border-color: #000000;
            }
            hr {
                width: 75%;
            }
        </style>
        <t t-set="now" t-value="datetime.datetime.now()"/>
        <div class="s_header_text_social o_mail_block_header_text_social o_mail_snippet_general pt16 pb16" data-snippet="s_mail_block_header_text_social" data-name="Left Text">
            <div class="container">
                <div class="row">
                    <div class="col-lg-4 pt16 pb16">
                        <h1>
                            <a t-att-href="(company_id.website) or '#'" target="_blank">
                                <span style="color: rgb(255, 187, 0) !important; font-size: 32px" class="fa fa-1x fa-sun-o" />
                            </a><span style="font-size: 32px;"><font style="font-weight: bolder; color: rgb(255, 255, 255); background-color: rgb(255, 187, 0);">SOLAR</font></span>
                        </h1>
                    </div>
                    <div class="col-lg-8 o_mail_header_social" style="text-align:right;">
                        <a href="/view">
                            <span style="font-weight: bolder;">View online</span>&amp;nbsp;<span class="fa fa-external-link" />
                        </a>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_title o_mail_snippet_general pt72 pb80" data-snippet="s_title" data-name="Title" style="background-color: rgb(255, 187, 0) !important;">
            <div class="container s_allow_columns">
                <h1 style="text-align:center">
                        <font style="font-size: 62px; background-color: rgb(255, 255, 255);">&amp;nbsp;10% OFF&amp;nbsp;</font>
                    <br/>
                        <font style="font-size: 62px; background-color: rgb(255, 255, 255);">&amp;nbsp;SALES&amp;nbsp;</font>
                </h1>
            </div>
        </div>
        <div class="s_discount2 o_mail_block_discount2 o_mail_snippet_general pt32 pb32" data-name="Promo Code" style="padding-left: 15px; padding-right: 15px;">
            <h2 style="text-align: center;">GET 10%&amp;nbsp;OFF</h2>
            <p style="text-align: center;">
                Valid on all sales prices in our webshop.
            </p>
            <table border="0" cellpadding="0" cellspacing="0" align="center" class="border" style="border-collapse: collapse; border-color: rgb(255, 187, 0) !important; border-width: 3px !important; mso-table-lspace:0pt; mso-table-rspace:0pt;">
                <tr>
                    <td width="50" height="50" align="center" class="o_mail_no_resize o_cc" style="width:50px!important; min-width: 50px; max-width:5.6rem; text-align: center"><i class="fa fa-2x fa-ticket" style="color: rgb(255, 187, 0) !important;"></i></td>
                    <td width="200" height="50" align="center" class="o_cc" style="min-width: 150px; width: 200px; text-align: center;"><p class="mb0">ENDOFSUMMER20</p></td>
                </tr>
            </table>
            <br/>
            <p style="text-align:center;">
                <a role="button" class="btn btn-primary btn-lg" href="http://">Visit the website</a>
            </p>
        </div>
        <div class="s_hr pt16 pb16" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <div class="s_features o_mail_snippet_general" data-snippet="s_features" data-name="Features">
            <div class="container">
                <div class="row">
                    <div class="col-lg-4 pt16 pb16" style="text-align:center;">
                        <img alt="Customer Service" src="/mass_mailing_themes/static/src/img/theme_promotion/s_default_image_product_1.jpg" class="img-fluid o_we_custom_image mx-auto d-block" style="width: 100px;"/>
                        <h3>24/7 <br/>Customer Service</h3>
                    </div>
                    <div class="col-lg-4 pt16 pb16" style="text-align:center;">
                        <img alt="Free Delivery" src="/mass_mailing_themes/static/src/img/theme_promotion/s_default_image_product_2.jpg" class="img-fluid o_we_custom_image mx-auto d-block" style="width: 100px;"/>
                        <h3 style="text-align:center;">Free delivery*</h3>
                        <p style="text-align:center;">*For orders over $39.</p>
                    </div>
                    <div class="col-lg-4 pt16 pb16" style="text-align:center;">
                        <img alt="Easy Returns" src="/mass_mailing_themes/static/src/img/theme_promotion/s_default_image_product_3.jpg" class="img-fluid o_we_custom_image mx-auto d-block" style="width: 100px;"/>
                        <h3 style="text-align:center;">Easy Returns</h3>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_left o_mail_snippet_general pt16 pb16" data-snippet="s_mail_block_footer_social_left" data-name="Footer Left" style="background-color: rgb(255, 187, 0) !important;">
            <div class="container">
                <div class="row">
                    <div class="col-lg-6">
                        <p>
                            <span style="font-weight: bolder;">
                                <font style="background-color: rgb(255, 255, 255); color: rgb(255, 187, 0);">SOLAR</font>
                            </span>
                        </p>
                        <div class="o_mail_footer_links">
                            <a role="button" href="/unsubscribe_from_list" class="btn btn-link ">
                                Unsubscribe
                            </a>
                        </div>
                    </div>
                    <div class="col-lg-6">
                        <div class="pb16" style="text-align: right;" t-translation="off">
                            <a aria-label="Facebook" title="Facebook" href="https://www.facebook.com/Odoo">
                                <span class="fa fa-facebook bg-black" style="color: rgb(255, 187, 0) !important;"/>
                            </a>&amp;nbsp;&amp;nbsp;
                            <a style="margin-left:10px" aria-label="LinkedIn" title="LinkedIn" href="https://www.linkedin.com/company/odoo">
                                <span class="fa fa-linkedin bg-black" style="color: rgb(255, 187, 0) !important;"/>
                            </a>&amp;nbsp;&amp;nbsp;
                            <a style="margin-left:10px" aria-label="Twitter" title="Twitter" href="https://twitter.com/Odoo">
                                <span class="fa fa-twitter bg-black" style="color: rgb(255, 187, 0) !important;"/>​
                            </a>&amp;nbsp;&amp;nbsp;
                            <a aria-label="Instagram" title="Instagram" href="https://www.instagram.com/explore/tags/odoo/">
                                <span class="fa fa-instagram bg-black" style="color: rgb(255, 187, 0) !important;"/>
                            </a>&amp;nbsp;&amp;nbsp;
                            <a aria-label="TikTok" title="TikTok" href="https://www.tiktok.com/@odoo">
                                <span class="fa fa-tiktok bg-black" style="color: rgb(255, 187, 0) !important;"/>
                            </a>
                        </div>
                        <p style="text-align: right">
                            <font style="color: rgb(137, 101, 1);">© <t t-out="now.strftime('%Y')" /> All Rights Reserved</font>
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Theme "CoffeeBreak" default template -->
    <template id="theme_coffeebreak_template">
        <style id="design-element">
            h1 {
                font-size: 42px;
                font-weight: bolder;
                color: #543427;
                font-family: Georgia, Times, "Times New Roman", serif;
            }
            h2 {
                font-size: 14px;
                color: #937853;
            }
            h3 {
                font-size: 14px;
                font-weight: bolder;
                color: #543427;
            }
            a:not(.btn), a.btn.btn-link {
                text-decoration-line: underline;
                color: #397B21;
            }
            a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary {
                color: #543427;
                background-color: #D3C5B1;
                border-color: #D3C5B1;
            }
            a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary {
                color: #FFFFFF;
                background-color: #543427;
                border-color: #543427;
            }
            hr {
                background-color: rgba(0,0,0,0); 
            }
        </style>
        <div class="s_header_social o_mail_block_header_social o_mail_snippet_general" style="background-color: rgb(238, 233, 226) !important;" data-snippet="s_mail_block_header_social" data-name="Left Logo">
            <div class="container">
                <div class="row">
                    <div class="col-lg-8 pt16 pb16">
                        <a style="text-decoration:none;float:none;" href="http://www.example.com">
                             <img border="0" src="/mass_mailing_themes/static/src/img/theme_coffeebreak/s_default_image_logo.png" style="height:auto;max-width:400px; width:100px;" alt="Your Logo" class="img-fluid" />
                        </a>
                    </div>
                    <div class="col-lg-4 o_mail_header_social" style="text-align: right">
                            <a aria-label="Facebook" title="Facebook" target="_blank" href="https://www.facebook.com/Odoo">
                                <span class="fa fa-facebook" style="color: rgb(84, 52, 39) !important; background-color: rgb(211, 197, 177) !important;">​</span>
                            </a>&amp;nbsp;&amp;nbsp;
                            <a style="margin-left:10px" aria-label="Twitter" title="Twitter" target="_blank" href="https://twitter.com/Odoo">
                                <span class="fa fa-twitter" style="color: rgb(84, 52, 39) !important; background-color: rgb(211, 197, 177) !important;">​</span>
                            </a>&amp;nbsp;&amp;nbsp;
                            <a style="margin-left:10px" aria-label="Instagram" title="Instagram" target="_blank" href="https://www.instagram.com/explore/tags/odoo/" contenteditable="true">
                                <span class="fa fa-instagram" style="color: rgb(84, 52, 39) !important; background-color: rgb(211, 197, 177) !important;">​</span>
                            </a>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_title o_mail_snippet_general pt16 pb16" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <h1 class="text-center">Company News</h1>
                <h2 class="text-center">28 Jan 2022</h2>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt16 pb0" style="background-color: rgb(238, 233, 226) !important;" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid" style="border-color: rgba(0,0,0,0)"/>
        </div>
        <div class="s_cover o_mail_snippet_general" data-snippet="s_cover" data-name="Cover">
            <img src="/mass_mailing_themes/static/src/img/theme_coffeebreak/s_default_image_block_banner.jpg" alt="Cover image" class="img-fluid w-100 mx-auto"/>
        </div>
        <div class="s_hr o_mail_snippet_general pt16 pb0" style="background-color: rgb(238, 233, 226) !important;" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid" style="border-color: rgba(0,0,0,0)"/>
        </div>
        <div class="s_text_block o_mail_snippet_general pt40 pb40 px-3" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <div class="row">
                    <div class="col-lg-5 offset-lg-1">
                        <p>If you didn't gather round the coffee machine or water-tank with your colleagues this week,
                        you've probably missed some key information.</p>
                        <p>We're here to make sure you miss none of the gossip.</p>
                    </div>
                    <div class="col-lg-4 offset-lg-1 pt24">
                        <blockquote>
                            <p>Communication is the key to success in any business.</p>
                            <p style="text-align: right;">
                                <span style="font-size: 12px; font-style: normal; font-weight: bolder;">- Lee Brown</span>
                            </p>
                        </blockquote>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt16 pb0" style="background-color: rgb(238, 233, 226) !important;" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid" style="border-color: rgba(0,0,0,0)"/>
        </div>
        <div class="s_text_image o_mail_snippet_general pt32 pb32" data-snippet="s_image_text" data-name="Image - Text">
            <div class="container">
                <div class="row align-items-center">
                    <div class="col-lg-10 offset-lg-1 px-0">
                        <h3>NEW PEOPLE</h3>
                        <img src="/mass_mailing_themes/static/src/img/theme_coffeebreak/s_default_image_block_image_text.jpg" class="img w-100"/>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-10 offset-lg-1 pt16 pb0">
                        <p>We're super excited to present our three new members, freshly arrived this week in our offices.</p>
                        <p><span style="font-weight: bolder;">Mark</span> will be helping our beloved HR team, he'll be taking care of your payroll, so be nice!</p>
                        <p>You'll be seeing <span style="font-weight: bolder;">Armando</span> around the kitchen since he's our new <span style="font-weight: bolder;">chef's assistant</span>!</p>
                        <p><span style="font-weight: bolder;">Martin</span> is currently doing his onboarding in our remote office but you'll be seeing him in the R&amp;D department very soon.</p>
                        <p style="text-align: center; font-size: 14px;"><strong>Give them a warm welcome!</strong></p>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt16 pb0" style="background-color: rgb(238, 233, 226) !important;" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid" style="border-color: rgba(0,0,0,0)"/>
        </div>
        <div class="s_text_block o_mail_snippet_general pt40 pb32" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <div class="row">
                    <div class="offset-lg-1 col-lg-10">
                        <h3>DON'T MISS THIS</h3>
                        <ul>
                            <li>
                                <a href="#" target="_blank" data-original-title="" title="">10 Books To Read in 2022</a>
                            </li>
                            <li>
                                <a href="#" target="_blank" data-original-title="" title="">Find Out Who's Employee Of The Month</a>
                            </li>
                            <li>
                                <a href="#" target="_blank" data-original-title="" title="">Why You Need Two Types of Content Strategists</a>
                            </li>
                            <li>
                                <a href="#" target="_blank" data-original-title="" title="">7 Ways to Nurture Creativity</a>
                            </li>
                            <li>
                                <a href="#" target="_blank" data-original-title="" title="">Infographic: Our Company Throughout The Generations</a>
                            </li>
                            <li>
                                <a href="#" target="_blank" data-original-title="" title="">7 Unexpected Signs You're Good At Your Job</a>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt16 pb0" style="background-color: rgb(238, 233, 226) !important;" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid" style="border-color: rgba(0,0,0,0)"/>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general o_cc o_cc5 pt24 pb24" style="background-color: rgb(84, 52, 39) !important;" data-snippet="s_mail_block_footer_social" data-name="Footer Center">
            <div class="container">
                <div class="row">
                    <div class="col-10 offset-lg-1">
                        <div class="o_mail_footer_social p16">
                            <a aria-label="Facebook" title="Facebook" target="_blank" href="https://www.facebook.com/Odoo"><span class="fa fa-facebook text-o-color-4"></span></a> &amp;nbsp;&amp;nbsp;
                            <a style="margin-left:10px" aria-label="Twitter" target="_blank" title="Twitter" href="https://twitter.com/Odoo"><span class="fa fa-twitter text-o-color-4"></span></a>  &amp;nbsp;&amp;nbsp;
                            <a style="margin-left:10px" aria-label="Instagram" title="Instagram" target="_blank" href="https://www.instagram.com/explore/tags/odoo/"><span class="fa fa-instagram text-o-color-4"></span></a>
                        </div>
                        <div class="o_mail_footer_links">
                            <a role="button" class="text-o-color-4" href="/unsubscribe_from_list">Unsubscribe</a> | <a role="button" class="text-o-color-4" href="/contactus">Contact</a>
                        </div>
                        <p class="o_mail_footer_copy text-o-color-4">
                            © <t t-esc="datetime.datetime.now().year"/> All Rights Reserved
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Theme "Newsletter" default template -->
    <template id="theme_newsletter_template">
        <style id="design-element">
            h1 {
                color: #CE0000;
                font-size: 54px;
            }
            h2{
                font-weight: bolder;
            }
            h3 {
                font-size: 16px;
                font-weight: bolder;
            }
            p, p > *, li, li > * {
                font-size: 16px;
            }
            a:not(.btn), a.btn.btn-link {
                text-decoration-line: underline;
                color: #212529;
            }
            a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary {
                background-color: #CE0000;
                border-color: #CE0000;
            }
            a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary {
                color: #CE0000;
                background-color: #FFFFFF;
                border-color: #CE0000;
            }
        </style>
        <div class="o_snippet_view_in_browser o_mail_snippet_general pt16 pb0" style="text-align: left; padding-left: 15px; padding-right: 15px; color: rgb(183, 191, 199) !important" data-snippet="s_mail_block_header_view" data-name="View Online"><a href="/view" class="" target="_blank" data-bs-original-title="" title="">View Online</a> - <span style="color: rgb(183, 191, 199); font-size: 12.25px" t-out="datetime.datetime.now().strftime('%B %d %Y')"></span></div>
        <div class="s_title o_mail_snippet_general pt32 pb32" data-snippet="s_title" data-name="Title" style="background-color: rgb(255, 255, 255) !important;">
            <div class="container s_allow_columns">
                <h1 style="text-align:center" class="">Weekly Musings</h1>
            </div>
        </div>
        <div class="s_title o_mail_snippet_general px-3 pb0 pt0" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                    <h2 style="" class="">The story of Odoo</h2>
            </div>
        </div>
        <div class="s_header_social o_mail_block_header_social o_mail_snippet_general bg-white px-3 pt24 pb24" style="" data-snippet="s_mail_block_header_social" data-name="Left Logo">
            <div class="container" style="">
                <div class="col-lg-12 o_mail_header_social" style="background-color: rgb(255, 255, 255) !important; text-align: left;">
                    <a aria-label="Facebook" target="_blank" href="https://www.facebook.com/Odoo" data-original-title="Facebook" rel="noreferrer">
                        <img class="img-fluid o_we_custom_image" src="/mass_mailing_themes/static/src/img/theme_newsletter/social-facebook.png" data-original-id="435" data-original-src="/mass_mailing_themes/static/src/img/theme_newsletter/social-facebook.png" data-mimetype="image/png" style="width: 30px;"/>
                    </a>&amp;nbsp;&amp;nbsp;
                    <a aria-label="Twitter" target="_blank" href="https://twitter.com/Odoo" data-original-title="Twitter" rel="noreferrer">
                        <img class="img-fluid o_we_custom_image" src="/mass_mailing_themes/static/src/img/theme_newsletter/social-twitter.png" data-original-id="436" data-original-src="/mass_mailing_themes/static/src/img/theme_newsletter/social-twitter.png" data-mimetype="image/png" style="width: 30px;"/>
                    </a>&amp;nbsp;&amp;nbsp;
                    <a  aria-label="Instagram" target="_blank" href="https://www.linkedin.com/company/odoo" data-original-title="Linkedin" rel="noreferrer">
                        <img class="img-fluid o_we_custom_image" src="/mass_mailing_themes/static/src/img/theme_newsletter/social-linkedin.png" data-original-id="437" data-original-src="/mass_mailing_themes/static/src/img/theme_newsletter/social-linkedin.png" data-mimetype="image/png" style="width: 30px;"/>
                    </a>
                </div>
            </div>
        </div>
         <div class="s_text_block o_mail_snippet_general px-3 pb24" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p style="font-size: 14px;" class="small text-muted">Musings on the week from <a style="font-size: 14px;" href="https://twitter.com/fpodoo?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor">@fpodoo</a></p>
            </div>
        </div>
        <div class="s_title o_mail_snippet_general px-3 pb0 pt0" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <h3>Making companies a better place, one app at a time</h3>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt16 pb24 " data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>I needed to change the world. I wanted to... You know how it is when you are young; you have big dreams, a lot of energy and naive stupidity. My dream was to lead the enterprise management market with a fully open source software (I also wanted to get 100 employees before 30 years old with a self-financed company but I failed this one by only a few months).</p>
                <p>To fuel my motivation, I had to pick someone to fight against. In business, it's like a playground. When you arrive in a new school, if you want to quickly become the leader, you must choose the class bully, the older guy who terrorizes small boys, and kick his butt in front of everyone. That was my strategy with SAP, the enterprise software giant.</p>
                <p>So in 2005, I started to develop the TinyERP product, the software that (at least in my mind) would change the enterprise world. While preparing for the "day of the fight" in 2006, I <a href="http://whois.domaintools.com/sorrysap.com" target="_blank">bought the SorrySAP.com domain name</a>. I put it on hold for 6 years, waiting for the right moment to use it. I thought it would take 3 years to deprecate a 77 billion dollars company just because open source is so cool. Sometimes it's better for your self-motivation not to face reality...</p>
                <p>To make things happen, I worked hard, very hard. I worked 13 hours a day, 7 days a week, with no vacation for 7 years. I lost friendships and broke up with my girlfriend in the process.</p>
                <p>Three years later, I discovered you can't change the world if you are "tiny". Especially if the United States is part of this world, where it's better to be a BigERP, rather than a TinyERP. Can you imagine how small you feel <a href="http://slidesha.re/ohc44E" target="_blank">in front of Danone's directors</a> asking; "But why should we pay millions of dollars for a tiny software?" So, we renamed TinyERP to OpenERP.</p>
                <p>As we worked hard, things started to evolve. We were developing dozens of modules for OpenERP, the open source community was growing and I was even able to pay all employees' salaries at the end of the month without fear (which was a situation I struggled with for 4 years).</p>
                <p>In 2010, we had a 100+ employees selling services on OpenERP and a powerful but ugly product. This is what happens when delivering services to customers distracts you from building an exceptional product. It was time to do a pivot in the business model.</p>
            </div>
        </div>
        <div class="s_cover o_mail_snippet_general" data-snippet="s_cover" data-name="Cover">
            <img src="/mass_mailing_themes/static/src/img/theme_newsletter/s_default_image_block_banner.jpg" alt="Cover image" class="img-fluid w-100 mx-auto"/>
        </div>
        <div class="s_title o_mail_snippet_general px-3 pb0 pt0" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <h3>The Pivot</h3>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt16 pb24" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>We wanted to switch from a service company to a software publisher company. This would allow us to increase our efforts in our research and development activities. As a result, <a href="https://accounts.openerp.com/blog/OpenERP-Blog-Post-1/post/Open-Source-Free-as-in-Freedom-not-free-as-in-Free-services-119" target="_blank">we changed our business model</a> and decided to stop our services for customers and focus on building a strong partner network and maintenance offer. This would cost money, so I had to raise a few million euros.</p>
                <p>After a few months of pitching investors, I got roughly 10 LOI from different VCs. We chose Sofinnova Partners, the biggest European VC, and Xavier Niel the founder of Iliad, the only company in France funded in the past 10 years to have reached the 1 billion EUR valuation.</p>
                <p>I signed the LOI. I didn't realize that this contract could have turned me into a homeless person. (I already had a dog, all I needed was to lose a lot of money to become homeless). The fundraising was based on a company valuation but there was a financial mechanism to re-evaluate the company up by 9.8 m€ depending on the turnover of the next 4 years. I should have received warrants convertible into shares if we achieved the turnover targeted in the business plan.</p>
                <p>The night before receiving the warrants in front of the notary, my wife checked the contracts. She asked me what would be the taxation on these warrants. I rang the lawyer and guess what? Belgium is probably the only country in the world where you have to pay taxes on warrants when you receive them, even if you never reach the conditions to convert them into shares. If I had accepted these warrants, I would have had to pay a 12.5% tax on 9.8 m€; resulting in a tax of 1.2m€ to pay in 18 months! So, my wife is worth 1.2 million EUR. I would have ended up a homeless person without her, as I still did not have a salary at that time.</p>
                <p>We changed the deal and I got the 3 million EUR. It allowed me to recruit a rocking management team.</p>
            </div>
        </div>
        <div class="s_cover o_mail_snippet_general pb24" data-snippet="s_cover" data-name="Cover">
            <img src="/mass_mailing_themes/static/src/img/theme_newsletter/s_default_image_block_banner_2.jpg" alt="Cover image" class="img-fluid w-100 mx-auto"/>
        </div>
        <div class="s_title o_mail_snippet_general px-3 pb0 pt0" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <h3>Being a Mature Company</h3>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt16 pb24" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>With this money in our bank account, we boosted two departments: R&amp;D and Sales. We burned 2 million EUR in 18 months, mostly in salaries. The company started to grow even faster. We developed a partner network of 500 partners in 100 countries and we started to sign contracts with 6 zeros.</p>
                <p>Then, things became different. You know, tedious things like handling human resources, board meetings, dealing with big customer contracts, traveling to launch international subsidiaries. We did boring stuff like budgets, career paths, management meetings, etc.</p>
                <p>2011 was a complex year. We did not meet our expectations: we only achieved 70% of the forecasted sales budget. Our management meetings were tense. We under-performed. We were not satisfied with ourselves. We had a constant feeling that we were missing something. It's a strange feeling to build extraordinary things but to not be proud of ourselves.</p>
                <p>But one day, someone (I don't remember who, I have a goldfish memory) made a graph of the monthly turnover of the past 2 years. It was like waking up from a nightmare. In fact, it was not that bad, we had multiplied the monthly turnover by 10 over the span of roughly two years! This is when we understood that OpenERP is a marathon, not a sprint. Only 100% growth a year is ok... if you can keep the rhythm for several years. </p>
                <p>As usual, I should have listened to my wife. She is way more lucid than I am. Every week I complained to her "It's not good enough, we should grow faster, what am I missing?" and she used to reply: "But you already are the fastest growing company in Belgium!" (Deloitte awarded us as <a href="https://accounts.openerp.com/blog/OpenERP-Blog-1/post/Three-Awards-for-OpenERP-in-2013-139" target="_blank">the fastest growing company of Belgium</a> with 1549% growth of the turnover between 2007 and 2011).</p>
            </div>
        </div>
        <div class="s_title o_mail_snippet_general px-3 pb0 pt0" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <h3>Changing the World</h3>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt16 pb24" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>Then the dream started to become reality. We started to get clues that what we did would change the world:</p>
                <ul>
                    <li>With 1000 installations per day, we became the most installed management software in the world
                    </li>
                    <li><a href="http://amzn.to/P4lRmE" target="_blank">Analysts from Big 4 started to prefer OpenERP over SAP</a></li>
                    <li>OpenERP is now a compulsory subject for the baccalaureate in France like Word, Excel and PowerPoint</li>
                    <li>60 new modules are released every month (we became the wikipedia of the management software thanks to our strong community)</li>
                    <li>In 2013, we had 2,000,000 users worldwide</li>
                </ul>
                <p>Something is happening... And it's big!</p>
            </div>
        </div>
        <div class="s_title o_mail_snippet_general px-3 pb0 pt0" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <h3>New Financing in 2014</h3>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt16 pb24" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>After years of rapid growth, we achieved another amazing goal - we secured a new round of 10 Million USD of financing, jointly provided by leading venture capital firms XAnge (France), SRIW (Belgium), Sofinnova (France), and the management team.</p>
                <p>The new financing will support the acceleration of the outstanding growth the company has seen over the last years, enabling the doubling of the commercial force and increased R&amp;D staff - already 100+ people strong. </p>
            </div>
        </div>
        <div class="s_cover o_mail_snippet_general pb24" data-snippet="s_cover" data-name="Cover">
            <img src="/mass_mailing_themes/static/src/img/theme_newsletter/s_default_image_block_banner_3.jpg" alt="Cover image" class="img-fluid w-100 mx-auto"/>
        </div>
        <div class="s_title o_mail_snippet_general px-3 pb0 pt0" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <h3>The Rise of Odoo</h3>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt16" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>After having disrupted the ERP market, OpenERP moved far beyond the boundaries of traditional ERP players. The integration of business activities is no longer restricted to sales, accounting, inventory and procurements. In June 2014, we released version 8 with awesome CMS &amp; eCommerce, a Point of Sale, an integrated Business Intelligence engine and much more (3000+ modules).</p>
                <p>Our exceptional technology allowed us to move to newer markets (CMS &amp; eCommerce) and propose a product that disrupts existing open source players (Wordpress, Magento, etc). The OpenERP CMS is so good that even top competitors admitted it publicly when we released the beta version.</p>
                <p>If current open source players have a real technical challenge to follow us, we also have a huge marketing challenge. Wordpress has 22% of all internet websites. We were the leader in management software, but we start with 0% of the website market.</p>
                <p>To move forward, we raised 10 million USD in May 2014 in order to boost marketing and sales activities. In order to support our vision, we had to change the name that is not restricted to ERPs functions. We needed a name that allowed us to support our ambitions; build business solutions like CMS, eCommerce, Business Intelligence and, who knows, even sky rockets or driverless cars in the future...</p>
                <p>In May 2014, we renamed the company and product to Odoo.</p>
                <p>And a new story just started...</p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general pb16 px-3" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <img src="/mass_mailing_themes/static/src/img/theme_newsletter/FPsignature.gif" style="width: 125px" alt=""/>
                <p style="text-align:center;" class="pt32">
                    <a role="button" href="#" class="btn btn-primary btn-lg" target="_blank">&amp;nbsp;&amp;nbsp;Comment on this post&amp;nbsp;&amp;nbsp;</a>
                </p>
            </div>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general o_cc o_cc1 pt0 pb0" data-snippet="s_mail_block_footer_social" data-name="Footer Center">
            <div class="container">
                <div class="col-lg-12 o_mail_footer_social pt0 pb0" style="text-align: center;">
                    <a aria-label="Facebook" class="text-white" target="_blank" href="https://www.facebook.com/Odoo" data-original-title="Facebook" rel="noreferrer">
                        <img class="img-fluid o_we_custom_image" src="/mass_mailing_themes/static/src/img/theme_newsletter/social-facebook.png" data-original-id="435" data-original-src="/mass_mailing_themes/static/src/img/theme_newsletter/social-facebook.png" data-mimetype="image/png" style="width: 30px;"/>
                    </a>&amp;nbsp;&amp;nbsp;
                    <a class="text-white" aria-label="Twitter" target="_blank" href="https://twitter.com/Odoo" data-original-title="Twitter" rel="noreferrer">
                        <img class="img-fluid o_we_custom_image" src="/mass_mailing_themes/static/src/img/theme_newsletter/social-twitter.png" data-original-id="436" data-original-src="/mass_mailing_themes/static/src/img/theme_newsletter/social-twitter.png" data-mimetype="image/png" style="width: 30px;"/>
                    </a>&amp;nbsp;&amp;nbsp;
                    <a class="text-white" aria-label="Instagram" target="_blank" href="https://www.linkedin.com/company/odoo" data-original-title="Linkedin" rel="noreferrer">
                        <img class="img-fluid o_we_custom_image" src="/mass_mailing_themes/static/src/img/theme_newsletter/social-linkedin.png" data-original-id="437" data-original-src="/mass_mailing_themes/static/src/img/theme_newsletter/social-linkedin.png" data-mimetype="image/png" style="width: 30px;"/>
                    </a>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pt16 pb16" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid" style="border-top-color: rgb(108, 117, 125) !important;"/>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general o_cc o_cc1 pt0 pb0" data-snippet="s_mail_block_footer_social" data-name="Footer Center">
            <div class="container" style="text-align: center; color:rgb(183, 191, 199) !important;">
                <span style="font-size: 12px;" class="o_default_snippet_text"><font style="color: rgb(183, 191, 199);">250 Executive Park Blvd</font></span>
                <br/>
                <span style="font-size: 12px;" class=""><font style="color: rgb(183, 191, 199);">San Francisco, California 94134</font></span>
                <br/>
                <span style="font-size: 12px;" class="o_default_snippet_text"><font style="color: rgb(183, 191, 199);">You received this email from </font></span><a href="mailto:" target="_blank"><span style="font-size: 12px;" class="o_default_snippet_text"><font style="color: rgb(183, 191, 199);">hello@musingsoftheweek.com</font></span></a>
                <br/>
                <span style="font-size: 12px;" class="o_default_snippet_text"><font style="color: rgb(183, 191, 199);">To unsubscribe, </font></span><a href="/unsubscribe_from_list" target="_blank"><span style="font-size: 12px;" class="o_default_snippet_text"><font style="color: rgb(183, 191, 199);">click here</font></span></a><span style="font-size: 12px;" class="o_default_snippet_text"><font style="color: rgb(183, 191, 199);">.</font></span>
            </div>
        </div>
    </template>

    <!-- Roadshow 1 -->
    <template id="theme_roadshow1_template">
        <style id="design-element">
            h1 {
                font-size: 24px;
                font-weight: bolder;
            }
            h2 {
                font-size: 21px;
            }
            h3 {
                font-size: 18px;
                font-weight: bolder;
            }
            a:not(.btn), a.btn.btn-link {
                color: #35979c;
            }
        </style>
        <div class="s_text_block o_mail_snippet_general pt40 px-3 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <div class="s_title o_mail_snippet_general pb0 pt0" data-snippet="s_title" data-name="Title">
                    <div class="container s_allow_columns">
                        <h1>We're almost there!</h1>
                    </div>
                </div>
                <p>Odoo invites you on <span class="font-weight: bolder">[date]</span> at <span class="font-weight: bolder">[time]</span> for their infamous roadshow in <span class="font-weight: bolder">[city]</span>: an event that combines learning and networking in a casual afterwork-like atmosphere.</p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt8 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>Registration is free, but mandatory:</p>
                <p style="text-align: center;">
                    <a href="#" class="btn btn-primary btn-lg">Save your seat now</a>
                </p>
                <p style="text-align: center;">
                    <span class="font-weight: bolder">
                        <font style="color: rgb(206, 0, 0); font-weight: normal;">(only 195 tickets left)</font>
                    </span>
                </p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt0 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>Discover Odoo, the all-in-one management platform, and its apps designed to improve employees' daily tasks. From sales and deliveries, to logistics, marketing and accounting ; learn what are the best available options to digitize your company.</p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt0 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>[speaker] and [speaker] will rely on a unique demo concept in which YOU lead the demo and choose what you wish to see. They'll show how to tackle the challenge using the best tools, live. You will be amazed!</p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt0 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>You may then take part in round table discussions with experts who will give you their tips and tricks to help digitize companies. We'll wrap up the event with a networking session including a walking dinner.</p>
            </div>
        </div>
        <div class="s_title o_mail_snippet_general pb0 pt8" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <div class="row">
                    <div class="offset-lg-2 col-lg-8">
                        <div class="s_title o_mail_snippet_general pb0 pt0" data-snippet="s_title" data-name="Title">
                            <div class="container s_allow_columns">
                                <h3 class="text-center">Schedule</h3>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_hr o_mail_snippet_general pb16 pt4" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid w-50"/>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt8 pb32" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <table class="mx-auto">
                    <tr style="vertical-align:top">
                        <td style="text-align: right;" class="p-2">
                            <span class="font-weight: bolder">8h:</span>
                        </td>
                        <td class="p-2">
                            <span class="font-weight: bolder">Welcome</span>
                        </td>
                    </tr>
                    <tr style="vertical-align:top">
                        <td style="text-align: right;" class="p-2">
                            <span class="font-weight: bolder">18h30:</span>
                        </td>
                        <td class="p-2">
                            <span class="font-weight: bolder">[speaker] &amp; [speaker]'s conference</span>
                            <ul>
                                <li>Interactive Odoo demo</li>
                                <li>Best practices to digitize your company</li>
                            </ul>
                        </td>
                    </tr>
                    <tr style="vertical-align:top">
                        <td style="text-align: right;" class="p-2">
                            <span class="font-weight: bolder">19h00:</span>
                        </td>
                        <td class="p-2">
                            <span class="font-weight: bolder">Table Ronde</span>
                            <ul>
                                <li>Business Case</li>
                                <li>Client Testimonials</li>
                            </ul>
                        </td>
                    </tr>
                    <tr style="vertical-align:top">
                        <td style="text-align: right;" class="p-2">
                            <span class="font-weight: bolder">19h30: </span>
                        </td>
                        <td class="p-2">
                            <span class="font-weight: bolder">Networking: Walking Dinner &amp; Drinks</span>
                        </td>
                    </tr>
                </table>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt0 pb4" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>For more information, check the event page : <a href="#" target="_blank" rel="noreferrer">[link]</a>.</p>
                <p>We're on the starting blocks to prepare an incredible event for you!
                    <br/>We hope to see you there.
                </p>
                <p>See you soon.</p>
                <p>--</p>
                <p>[speaker] &amp; [speaker]</p>
                <p><span style="font-style: italic;"><font class="text-600">The first 20 participants will get a free copy of our business game "Scale-Up".</font></span></p>
            </div>
        </div>
    </template>

    <!-- Roadshow 2 -->
    <template id="theme_roadshow2_template">
        <style id="design-element">
            h1 {
                font-size: 24px;
                font-weight: bolder;
            }
            h2 {
                font-size: 21px;
            }
            h3 {
                font-size: 18px;
                font-weight: bolder;
            }
            a:not(.btn), a.btn.btn-link {
                color: #35979c;
            }
        </style>
        <div class="s_text_block o_mail_snippet_general px-3 pt40 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <div class="s_title o_mail_snippet_general pb0 pt0" data-snippet="s_title" data-name="Title">
                    <div class="container s_allow_columns">
                        <h1>Yesterday's event at [venue] in [city] was incredible!</h1>
                    </div>
                </div>
                <p>Here are some photos taken during the event:</p>
                <ul>
                    <li><a href="#" target="_blank">Link 1</a></li>
                    <li><a href="#" target="_blank">Link 2</a></li>
                </ul>
                <p>Don't hesitate to send us more!</p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt8 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>You had the chance to discover Odoo during a full 3 hours, how great is that?
                    <br/>Now that you've seen Odoo in action and asked some questions, you likely need more information to move forward in your project and to work with us. You'll find additional points just below.
                </p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt8 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <div class="s_title o_mail_snippet_general pb0 pt0" data-snippet="s_title" data-name="Title">
                    <div class="container s_allow_columns">
                        <h3>Do you wish to use Odoo?</h3>
                    </div>
                </div>
                <p>If you wish to evaluate Odoo while taking into account your company's specific needs, you can book a free appointment with one of our <a href="https://odoo.com/r/meeting" target="_blank">Odoo Experts</a> or <a href="https://odoo.com/page/tour" target="_blank">watch our videos about the product</a>.</p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt8 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>Here are the links to our sponsors' websites if you wish to meet them:</p>
                <ul>
                    <li><a href="#" target="_blank">Link 1</a></li>
                    <li><a href="#" target="_blank">Link 2</a></li>
                    <li><a href="#" target="_blank">Link 3</a></li>
                    <li><a href="#" target="_blank">Link 4</a></li>
                </ul>
                <p>You may test Odoo for free by clicking <a href="https://odoo.com/trial" target="_blank">here</a>.</p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt8 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <div class="s_title o_mail_snippet_general pb0 pt0" data-snippet="s_title" data-name="Title">
                    <div class="container s_allow_columns">
                        <h3>Do you wish to become a partner?</h3>
                    </div>
                </div>
                <p>If you wish to become a partner and offer your services in [country], take a look at <a href="https://odoo.com/r/meeting" target="_blank">this page</a>.</p>
                <p>Learn more about our pricing <a href="https://odoo.com/r/meeting" target="_blank">here</a>.</p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt8 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>You may then take part in round table discussions with experts who will give you their tips and tricks to help digitize companies. We'll wrap up the event with a networking session including a walking dinner.</p>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt8 pb0" data-snippet="s_text_block" data-name="Text">
            <div class="container s_allow_columns">
                <p>Thank you, once again, for your attendance at yesterday's event.</p>
                <p>--</p>
                <p>[speaker] &amp; [speaker]</p>
            </div>
        </div>
    </template>
    
    <!-- Training -->
    <template id="theme_training_template">
        <style id="design-element">
            * :not(.fa) {
                font-family: "Lucida Grande", "Lucida Sans Unicode", "Lucida Sans", Geneva, Verdana, sans-serif !important;
            }
            h1 {
                color: rgb(255, 255, 255);
                font-size: 39px;
            }
            h3 {
                color: rgb(20, 102, 184);
            }
            p, p > *, li, li > * {
                color: rgb(91, 119, 143);
                font-size: 14px;
            }
            a:not(.btn), a.btn.btn-link {
                color: rgb(20, 102, 184);
            }
            a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary {
                color: rgb(255, 255, 255);
                background-color: rgb(67, 128, 219);
                font-size: 14px;
            }
            a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary {
                color: rgb(255, 255, 255);
                background-color: rgb(255, 152, 0);
            }
            hr {
                border-top-color: rgb(67, 129, 219);
                border-top-width: 2px;
            }
        </style>
        <div class="s_header_logo o_mail_block_header_logo o_mail_snippet_general" data-snippet="s_mail_block_header_logo" data-name="Centered Logo" style="background-color: rgb(220, 234, 255);">
            <div class="container">
                <div class="row">
                    <div class="col-lg-4"></div>
                    <div class="col-lg-4 pt16 pb16" style="text-align: center;">
                        <a style="text-decoration:none;" href="#" target="_blank">
                            <font style="font-size: 24px; background-color: rgb(20, 102, 184);" class="text-o-color-4">A+</font> <font style="color: rgb(20, 102, 184); font-size: 24px;"><span style="font-weight: normal">Training</span></font>
                        </a>
                    </div>
                    <div class="col-lg-4" style="text-align: right;"></div>
                </div>
            </div>
        </div>
        <div class="s_picture o_mail_snippet_general pt48 px-3 o_cc o_cc2 pb0" data-snippet="s_picture" data-name="Picture" placeholder="Type &quot;/&quot; for commands" style="background-color: rgb(67, 128, 219);">
            <img class="img-fluid o_we_custom_image" src="/web_editor/image_shape/mass_mailing_themes.s_default_image_block_image/web_editor/composition/composition_organic_line.svg?c1=%23FF9800" data-shape="web_editor/composition/composition_organic_line" data-file-name="picture-composition_organic_line.svg" data-shape-colors="#FF9800;;;;" data-original-mimetype="image/jpeg"/>
        </div>
        <div class="s_title o_mail_snippet_general pt32 pb0" data-snippet="s_title" data-name="Title" style="background-color: rgb(67, 128, 219);">
            <div class="container s_allow_columns">
                <div class="row">
                    <div class="col-lg-10 offset-lg-1 pb32">
                        <h1>Train with the best developers</h1>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_mail_block_event o_mail_snippet_general bg-white" data-snippet="s_event" data-name="Event">
            <div class="container">
                <div class="row align-items-center">
                    <div class="s_col_no_bgcolor pt32 offset-lg-1 col-lg-10 pb0" style="padding-left: 0px; padding-right: 0px;">
                        <div class="card bg-white h-100 border" style="border-radius: 5px !important; border-color: rgb(233, 236, 239) !important; border-width: 0px !important;">
                            <div class="card-body">
                                <div class="s_title o_mail_snippet_general pb0 pt0" data-snippet="s_title" data-name="Title">
                                    <div class="container s_allow_columns">
                                        <h3 class="card-title">Software development training</h3>
                                    </div>
                                </div>
                                <p><span style="font-weight: bolder;">25 September 2022 - 4:30 PM</span></p>
                                <p><i class="fa fa-map-marker"/> London, United Kingdom​</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-10 offset-lg-1 pb0">
                        <p>Software developer training can be expensive, so we've compiled a list of free courses that are perfect for non-Computer Science graduates who want to upskill and learn to code.</p>
                        <p>Universities may give us a shiny degrees, but it does not give us the <span style="font-weight: bolder;">practical skills we need to start an innovative career in tech</span>.</p>
                        <p>Getting practical experience can be as likely as getting struck by lightning – <span style="font-style: italic;">thank you, coronavirus</span> – but, you can take things into your own hands and use your free time to upskill.</p>
                        <p><span style="font-weight: bolder;">And what better field to upskill in than software development?</span> Jobs have been steadily booming for years, and will continue to do so. By taking some of these free courses, you can make your CV more competitive and your portfolio more interesting.</p>
                        <p></p>
                        <p><a href="#" target="_blank" class="btn btn-primary flat btn-lg"><span style="font-weight: bolder;">Register now</span></a></p>
                    </div>
                </div>
            </div>
        </div>    
        <div class="container s_allow_columns">
            <div class="row">
                <div class="offset-lg-1 col-lg-10">
                    <p><img src="/mass_mailing_themes/static/src/img/theme_training/s_default_image_block_banner.jpg" class="img img-fluid o_we_custom_image" data-original-id="347" data-original-src="/mass_mailing_themes/static/src/img/theme_training/s_default_image_block_banner.jpg" data-mimetype="image/jpeg" style="width: 658px;"/></p>
                </div>
            </div>
        </div>
        <div class="s_cover o_mail_snippet_general" data-snippet="s_cover" data-name="Cover" style="background-color: rgb(220, 234, 255);">
            <div class="container">
                <div class="row">
                    <div class="oe_img_bg o_bg_img_center pb88 pt104" style="background-image: url('/mass_mailing_themes/static/src/img/theme_training/see_you_soon.svg');">
                        <div class="o_we_bg_filter" style="background-image: linear-gradient(0deg, rgb(67, 128, 219) 0%, rgba(67, 128, 219, 0.4) 100%);"/>
                        <h1 style="text-align: center;">See you soon !</h1>
                    </div>
                </div>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general px-3 pt16 pb0" data-snippet="s_text_block" data-name="Text" style="background-color: rgb(220, 234, 255);">
            <div class="container s_allow_columns">
                <p style="text-align: center">
                    <span style="font-size: 10px;">
                        <font style="color: rgb(112, 157, 225);" class="o_default_snippet_text">You are receiving this email because you showed an interest in our previous events
                        </font>
                    </span>
                </p>
                <div class="s_hr o_mail_snippet_general pt0 pb16" data-snippet="s_hr" data-name="Separator">
                    <hr class="s_hr_1px s_hr_solid w-50" contenteditable="false"/>
                </div>
            </div>
        </div>
        <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general" data-snippet="s_mail_block_footer_social" data-name="Footer Center" style="background-color: rgb(220, 234, 255); text-align: center;">
            <div class="container">
                <div class="row">
                    <div class="col-lg o_mail_footer_social pt8 pb8">
                        <a aria-label="Facebook" title="Facebook" href="https://www.facebook.com/Odoo">
                            <span class="fa fa-facebook" contenteditable="false">​</span>
                        </a>
                        <a style="margin-left:10px" aria-label="LinkedIn" title="LinkedIn" href="https://www.linkedin.com/company/odoo">
                            <span class="fa fa-linkedin" contenteditable="false">​</span>
                        </a>
                        <a style="margin-left:10px" aria-label="Twitter" title="Twitter" href="https://twitter.com/Odoo">
                            <span class="fa fa-twitter" contenteditable="false">​</span>
                        </a>
                        <a style="margin-left:10px" aria-label="Instagram" title="Instagram" href="https://www.instagram.com/explore/tags/odoo/">
                            <span class="fa fa-instagram" contenteditable="false">​</span>
                        </a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg o_mail_footer_links">
                        <a role="button" href="/unsubscribe_from_list" class="btn btn-link o_default_snippet_text">Unsubscribe</a>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg">
                        <p><span style="font-size: 10px;">
                            © 2022 All Rights Reserved
                        </span></p>
                    </div>
                </div>
            </div>
        </div>
    </template>
</odoo>

```

