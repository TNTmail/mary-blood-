import os
import sys
from PyQt6.QtWidgets import (
    QApplication, QWidget, QLabel, QVBoxLayout, QHBoxLayout, 
    QPushButton, QGridLayout, QGraphicsBlurEffect
)
from PyQt6.QtGui import QPixmap, QFont
from PyQt6.QtCore import Qt, QUrl
from PyQt6.QtMultimedia import QMediaPlayer, QAudioOutput

class AlbumViewer(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Mary's Blood")
        self.resize(1200, 700)
        
        # Base folder path
        self.folder_path = r"D:\mary"
        
        # Image list
        self.image_paths = [
            r"D:\mary\500x500-000000-80-0-0.jpg",
            r"D:\mary\500x500-000000-80-0-1.jpg",
            r"D:\mary\500x500-000000-80-0-2.jpg",
            r"D:\mary\500x500-000000-80-0-5.jpg",
            r"D:\mary\500x500-000000-80-0-9.jpg",
            r"D:\mary\500x500-000000-80-01.6.jpg",
            r"D:\mary\500x500-000000-80-0-45.jpg"
            
            
        ]
        self.current_index = 0

        # Audio Setup
        self.audio_output = QAudioOutput()
        self.player = QMediaPlayer()
        self.player.setAudioOutput(self.audio_output)
        self.audio_output.setVolume(0.8)

        # Scan self.folder_path for all MP3 files
        self.mp3_files = self.load_mp3_files()
        self.current_song_index = 0

        self.init_ui()

    def load_mp3_files(self):
        """Mary's Blood - World's End [Official Music video]."""
        mp3_list = []
        if os.path.exists(self.folder_path):
            for file in os.listdir(self.folder_path):
                if file.lower().endswith('.mp3'):
                    mp3_list.append(os.path.join(self.folder_path, file))
                    mp3_path = os.path.join(self.folder_path, file)
                    print(f"Found MP3: {mp3_path}")
        return mp3_list

    def init_ui(self):
        main_layout = QGridLayout(self)
        main_layout.setContentsMargins(0, 0, 0, 0)

        # Background grid
        bg_widget = QWidget(self)
        bg_grid = QGridLayout(bg_widget)
        bg_grid.setSpacing(0)
        bg_grid.setContentsMargins(0, 0, 0, 0)

        cols, rows = 12, 8
        img_count = len(self.image_paths)
        for r in range(rows):
            for c in range(cols):
                img_idx = (r * cols + c) % img_count
                lbl = QLabel()
                pix = QPixmap(self.image_paths[img_idx]).scaled(
                    120, 120, 
                    Qt.AspectRatioMode.KeepAspectRatioByExpanding, 
                    Qt.TransformationMode.SmoothTransformation
                )
                lbl.setPixmap(pix)
                lbl.setScaledContents(True)
                bg_grid.addWidget(lbl, r, c)

        blur = QGraphicsBlurEffect()
        blur.setBlurRadius(8)
        bg_widget.setGraphicsEffect(blur)
        main_layout.addWidget(bg_widget, 0, 0)

        overlay = QWidget(self)
        overlay.setStyleSheet("background-color: rgba(0, 0, 0, 0.65);")
        main_layout.addWidget(overlay, 0, 0)

        fg_widget = QWidget(self)
        fg_layout = QVBoxLayout(fg_widget)
        fg_layout.setAlignment(Qt.AlignmentFlag.AlignCenter)

        card = QWidget()
        card.setFixedSize(500, 620)
        card.setStyleSheet("""
            background-color: rgba(18, 18, 18, 0.9); 
            border-radius: 12px;
            border: 1px solid rgba(255, 255, 255, 0.1);
        """)
        
        card_layout = QVBoxLayout(card)
        card_layout.setContentsMargins(25, 20, 25, 20)

        self.title_label = QLabel("Mary's Blood")
        self.title_label.setFont(QFont("Arial", 18, QFont.Weight.Bold))
        self.title_label.setStyleSheet("color: white; border: none; background: transparent;")
        self.title_label.setAlignment(Qt.AlignmentFlag.AlignCenter)

        self.image_label = QLabel()
        self.image_label.setAlignment(Qt.AlignmentFlag.AlignCenter)
        self.image_label.setStyleSheet("border: none; background: transparent;")

        self.counter_label = QLabel()
        self.counter_label.setFont(QFont("Arial", 10))
        self.counter_label.setStyleSheet("color: #888888; border: none; background: transparent;")
        self.counter_label.setAlignment(Qt.AlignmentFlag.AlignCenter)

        self.song_label = QLabel("Spiller ingen sang")
        self.song_label.setFont(QFont("Arial", 10))
        self.song_label.setStyleSheet("color: #00ff66; border: none; background: transparent;")
        self.song_label.setAlignment(Qt.AlignmentFlag.AlignCenter)

        # Image navigation buttons
        btn_prev = QPushButton("◀ Forrige Billede")
        btn_prev.setStyleSheet("""
            QPushButton {
                color: white; background-color: #282828; 
                border: 1px solid #444; border-radius: 6px; padding: 8px 15px; font-weight: bold;
            }
            QPushButton:hover { background-color: #660E46; }
        """)
        btn_prev.clicked.connect(self.prev_image)

        btn_next = QPushButton("Næste Billede ▶")
        btn_next.setStyleSheet("""
            QPushButton {
                color: white; background-color: #282828; 
                border: 1px solid #444; border-radius: 6px; padding: 8px 15px; font-weight: bold;
            }
            QPushButton:hover { background-color: #383838; }
        """)
        btn_next.clicked.connect(self.next_image)

        # Audio Play/Pause Button
        self.btn_play_audio = QPushButton("▶ Afspil Musik")
        self.btn_play_audio.setStyleSheet("""
            QPushButton {
                color: white; background-color: #1b5e20; 
                border: 1px solid #2e7d32; border-radius: 6px; padding: 8px 15px; font-weight: bold;
            }
            QPushButton:hover { background-color: #2e7d32; }
        """)
        self.btn_play_audio.clicked.connect(self.toggle_audio)

        btn_layout = QHBoxLayout()
        btn_layout.addWidget(btn_prev)
        btn_layout.addWidget(btn_next)

        card_layout.addWidget(self.title_label)
        card_layout.addWidget(self.image_label, stretch=1)
        card_layout.addWidget(self.counter_label)
        card_layout.addWidget(self.song_label)
        card_layout.addLayout(btn_layout)
        card_layout.addWidget(self.btn_play_audio)

        fg_layout.addWidget(card)
        main_layout.addWidget(fg_widget, 0, 0)

        self.update_image()
        self.load_current_song()

    def load_current_song(self):
        """Loads the first found MP3 from self.folder_path."""
        if self.mp3_files:
            song_path = self.mp3_files[self.current_song_index]
            self.player.setSource(QUrl.fromLocalFile(song_path))
            song_name = os.path.basename(song_path)
            self.song_label.setText(f"Sang: {song_name}")
        else:
            self.song_label.setText("Ingen .mp3 fundet i mappen")

    def toggle_audio(self):
        """Plays or pauses the audio track."""
        if not self.mp3_files:
            return

        if self.player.playbackState() == QMediaPlayer.PlaybackState.PlayingState:
            self.player.pause()
            self.btn_play_audio.setText("▶ Afspil Musik")
        else:
            self.player.play()
            self.btn_play_audio.setText("⏸ Pause Musik")

    def update_image(self):
        pixmap = QPixmap(self.image_paths[self.current_index])
        scaled_pixmap = pixmap.scaled(
            380, 380, 
            Qt.AspectRatioMode.KeepAspectRatio, 
            Qt.TransformationMode.SmoothTransformation
        )
        self.image_label.setPixmap(scaled_pixmap)
        self.counter_label.setText(f"{self.current_index + 1} / {len(self.image_paths)}")

    def next_image(self):
        if self.current_index < len(self.image_paths) - 1:
            self.current_index += 1
            self.update_image()

    def prev_image(self):
        if self.current_index > 0:
            self.current_index -= 1
            self.update_image()

if __name__ == "__main__":
    app = QApplication(sys.argv)
    viewer = AlbumViewer()
    viewer.show()
    sys.exit(app.exec())
